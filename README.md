# Using variables with IBM Watson Conversation service
The [IBM Watson Conversation service](https://www.ibm.com/watson/developercloud/doc/conversation/) on [IBM Bluemix](http://www.ibm.com/cloud-computing/bluemix/) is the foundation for building powerful chatbots and dialog-based systems. Core to its flexibility and versatility is a programming API and the ability to pass variables through the cognitive system. It supports user-provided context variables, allows the access to identified intents and entities including the annotations (what was the system thinking). Moreover, it features predefined system entities which can be enabled and significantly simplify detection of numbers, locations, people, dates and more within the ongoing conversation.

In this repository we are going to collect samples that demonstrate how those variables and metadata can be put to productive work.

The samples and the general syntax are described in the following blog posts. They can serve as additional source of information:
 * [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)
 * [More Tips and Tricks for Building Chatbots](http://blog.4loeser.net/2017/06/more-tips-and-tricks-for-building.html)

# Samples
The following sample scenarios demonstrate how context variables, intents and entities including system entities can be used for customized dialogs.

* [Access to system entities](#access-to-system-entities)
* [Working with Currency](#working-with-currency)
* [Nested evaluation of variables](#nested-evaluation-of-variables)
* [Conditions and predicates in the response](#conditions-and-predicates-in-the-response)
* [Replaced Markers](#replaced-markers)
* [Random output from variables](#random-output-from-variables)
* [Zeroes at the Slots](#zeroes-at-the-slots)
* [Slack URIs as variable](#slack-uris-as-variables)

### Access to system entities
The IBM Watson Conversation service supports several system entities. They are predefined entities which can be enabled to allow simple identification of typical user input. The following response string shows how the entities can be accessed to form an answer.

Response string:
`All of <? entities.size() ?> entities: <? entities ?>. They include "sys-location" (<? entities['sys-location'] ?>), "sys-date" (<? entities['sys-date'] ?>), "sys-number" (<? entities['sys-number'] ?>), "sys-person" (<? entities['sys-person'] ?>) and more`

Input:   
`How is the weather today in San Francisco and in Germany. Will it be over 20? Henrik and family want to go outside`

Returned:
> All of 5 entities: San Francisco, Germany, Henrik, 2017-06-19, 20. They include "sys-location" (San Francisco, Germany), "sys-date" (2017-06-19), "sys-number" (20), "sys-person" (Henrik) and more

The example is part of the [`simple-variables.json`](workspaces/simple-variables.json) workspace file. You can import the file as new workspace into IBM Watson Conversation.

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

### Conditions and predicates in the response
Dialog nodes in IBM Watson Conversation allow to check conditions (or predicates). Conditions control whether the node qualifies. Conditions can also be placed ahead of a response block inside a dialog node. In addition to that, conditions can even be placed and evaluated inside individual responses.

The following is a reponse string. It checks whether the location of entities in the input string and then returns one of two strings. It uses condensed `ìf-then-else` logic, a so-called [ternary operator](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html):
```
<? entities['airport'][0].location[0]<entities['airport'][1].location[0] ?
'First, then second' : 'Second comes first' ?>
```

https://www.ibm.com/watson/developercloud/doc/conversation/expression-language.html

### Replaced Markers
An alternative to embedding expressions into responses is to use special markers in the regular text:   
```
Hello USER_NAME, it is CURRENT_TIME.
```

The application code would then search for and replace the designated markers with information carried in the application. This could be overall simpler to do, especially when more complex data and structures are retrieved from backend systems and more sophisticated formatting and processing of that data is needed.   
```
Hello Henrik, it is 16:54:31.
```
A simple example of the above can be seen in the [TJBot "Tell the time" recipe](https://github.com/damiancummins/tell_the_time). In the function [`tellTheTime`](https://github.com/DamianCummins/tell_the_time/blob/master/tellTheTime.js) the marker `todays_date` is replaced with the current time for a specific timezone.

In an advanced version of using markers, the entire response only consists of a marker. It would be replaced by the application by an answer that, maybe, is retrieved from a database or built from a template and data coming from various systems. The Watson Conversation Service would analyze and process the user input, but only return with a hint (the marker) on how to answer.   
```
PROVIDE_TIME
```
The above could be turned into:   
```
Hello Henrik, it is 16:54:31.
```
### Random output from variables
Sometimes, when using context variables or system metadata, you want to use a random (or other) portion of it in a response or when assigning a context variable. The following code shows how a mathematical function, here `random()`, is used to obtain an array entry and insert it into the response text:
```
Here is a random entity:
 <? entities[(entities.size() * T(java.lang.Math).random()).intValue()].value ?>
```
In the code the global entity array as part of the dialog metadata is used. A random number within the range of the array size is computed. The entity value is then accessed and printed.

### Zeroes at the Slots
[Slots](https://console.bluemix.net/docs/services/conversation/dialog-build.html#slots) are a great way to gather several pieces of information and to easily declare what is needed. However, sometimes "ease of use" and (mathematical) logic are not friendly to each other. The following is such an example: [Waiting for `@sys-number` to accept a `0` (zero)](https://stackoverflow.com/questions/45302644/ibm-watson-slots-wont-accept-0)    

The condition `@sys-number` is in fact a short hand syntax for condition `entities['sys-number'].value`. When `0` is sent the condition is evaluated to `false` as `0` is treated as a `false` by the expression language evaluator in Watson Conversation Service. Now this is not a desired behavior in this case. To prevent this, you can use `entities['sys-number']` in the condition that will return `true` every time `@sys-number` entity is recognized in the input. The [WCS documentation has some related usage tips](https://console.bluemix.net/docs/services/conversation/system-entities.html#sys-number).

### Slack URIs as variables
Lately, I have been using Slack integrations with Watson Conversation. One is the [Conversation connector](https://github.com/watson-developer-cloud/conversation-connector). When trying to retrieve and identify URIs, e.g. web addresses or email addresses, coming from Slack as a [patterned entity in Watson Conversation](https://console.bluemix.net/docs/services/conversation/entities.html#creating-entities), I initially failed. The reason is that Slack formats the URIs into a style like `<https://blog.4loeser.net|blog.4loeser.net>`. Thus, I am using the following pattern to define email and web address entities: `<[A-Za-z0-9._|:%@\/]+>`.


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
