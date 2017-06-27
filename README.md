# Using variables with IBM Watson Conversation service
The [IBM Watson Conversation service](https://www.ibm.com/watson/developercloud/doc/conversation/) on [IBM Bluemix](http://www.ibm.com/cloud-computing/bluemix/) is the foundation for building powerful chatbots and dialog-based systems. Core to its flexibility and versatility is a programming API and the ability to pass variables through the cognitive system. It supports user-provided context variables, allows the access to identified intents and entities including the annotations (what was the system thinking). Moreover, it features predefined system entities which can be enabled and significantly simplify detection of numbers, locations, people, dates and more within the ongoing conversation.

In this repository we are going to collect samples that demonstrate how those variables and metadata can be put to productive work.

The samples and the general syntax are described in the following blog posts. They can serve as additional source of information:
 * [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)
 * [More Tips and Tricks for Building Chatbots](http://blog.4loeser.net/2017/06/more-tips-and-tricks-for-building.html)

# Samples
The following sample scenarios demonstrate how context variables, intents and entities including system entities can be used for customized dialogs.

### Access to system entities
The IBM Watson Conversation service supports several system entities. They are predefined entities which can be enabled to allow simple identification of typical user input. The following response string shows how the entities can be accessed to form an answer.

Response string:
`All of <? entities.size() ?> entities: <? entities ?>. They include "sys-location" (<? entities['sys-location'] ?>), "sys-date" (<? entities['sys-date'] ?>), "sys-number" (<? entities['sys-number'] ?>), "sys-person" (<? entities['sys-person'] ?>) and more`

Input:   
`How is the weather today in San Francisco and in Germany. Will it be over 20? Henrik and family want to go outside`

Returned:
> All of 5 entities: San Francisco, Germany, Henrik, 2017-06-19, 20. They include "sys-location" (San Francisco, Germany), "sys-date" (2017-06-19), "sys-number" (20), "sys-person" (Henrik) and more


### Working with Currency
In this example a condition first checks that a system entity `sys-currency` is present, then that exactly two currency values are included in the user input and, as third condition, that the currency units match. In the response string the two entered values are added.

Condition:   
`entities['sys-currency'] && entities['sys-currency'].size()==2 &&  entities['sys-currency'][0].unit==entities['sys-currency'][1].unit`

Response string:
`You have <? entities['sys-currency'][0].value.toDouble() + entities['sys-currency'][1].value.toDouble() ?> <? @sys-currency.unit ?>`

Input:   
`I have 20 USD and $24`

Returned:
> You have 54.0 USD

### Nested evaluation of variables
Sometimes it is necessary to first evaluate one variable and then use the result as input to another evaluation. In some programming / scripting language there is a special method or operator for this. An example is the [`eval` function](https://docs.python.org/3.5/library/functions.html#eval) in Python. But how can you do something like this in Watson Conversation? There, evaluations are done inside the [`<? ?>` expression](https://www.ibm.com/watson/developercloud/doc/conversation/expression-language.html#evaluation).


The upper dialog node consists of an empty response (output). Only the context is evaluated and a new value assigned to the variables `access` and `result` (based on a computation):
```
{
  "context": {
    "access": "<? $myproperties['one'] ?>",
    "result": "<? $myproperties['two'] * @sys-number ?>"
  },
  "output": {}
}
```

A subnode with the following response performs the second set of evaluations. It uses `access` and `result` as input.
`This is my nested response: <? context['access'] ?>. The result is <? context['result'] + context['result'] ?>.`

If the dialog input is `nested 8` and there is an existing context with `myproperties.one="one"` and `myproperties.two=2`, the following result is returned:   
> This is my nested response: one. The result is 32.

### Conditions / predicates in the response
Dialog nodes in IBM Watson Conversation allow to check conditions (or predicates). Conditions control whether the node qualifies. Conditions can also be placed ahead of a response block inside a dialog node. In addition to that, conditions can even be placed and evaluated inside individual responses.

The following is a reponse string. It checks whether the location of entities in the input string and then returns one of two strings. It uses condensed `ìf-then-else` logic, a so-called [ternary operator](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html):
```
<? entities['airport'][0].location[0]<entities['airport'][1].location[0] ?
'First, then second' : 'Second comes first' ?>
```

https://www.ibm.com/watson/developercloud/doc/conversation/expression-language.html

# Documentation and Resources
Here are some useful links to documentation and other resources:
* Watson Conversation service: https://www.ibm.com/watson/developercloud/doc/conversation/index.html
* Watson Conversation service, expression language: https://www.ibm.com/watson/developercloud/doc/conversation/expression-language.html
* API for Watson Conversation service: https://www.ibm.com/watson/developercloud/conversation/api/v1/#introduction
* Python SDK, Watson Developer Cloud: https://github.com/watson-developer-cloud/python-sdk
* Node SDK, Watson Developer Cloud: https://github.com/watson-developer-cloud/node-sdk
* Java SDK, Watson Developer Cloud: https://github.com/watson-developer-cloud/java-sdk

# License
See [LICENSE](LICENSE) for license information.

The samples are provided on a "as-is" basis and are un-supported. Use with care...

# Contribute / Contact Information
If you have found errors or some instructions are not working anymore, then please open an GitHub issue or, better, create a pull request with your desired changes.

You can find more tutorials and sample code at:
https://ibm-bluemix.github.io/
