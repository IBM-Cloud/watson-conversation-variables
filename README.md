# Using variables with IBM Watson Conversation service
The IBM Watson Conversation service on IBM Bluemix is the foundation for building powerful chatbots and dialog-based systems. Core to its flexibility and versatility is a programming API and the ability to pass variables through the cognitive system. It supports user-provided context variables, allows the access to identified intents and entities including the annotations (what was the system thinking). Moreover, it features predefined system entities which can be enabled and significantly simplify detection of numbers, locations, people, dates and more within the ongoing conversation.

In this repository we are going to collect samples that demonstrate how those variables and metadata can be put to productive work.

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

# Documentation and Resources
Here are some useful links to documentation and other resources:
* Watson Conversation service: https://www.ibm.com/watson/developercloud/doc/conversation/index.html
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
