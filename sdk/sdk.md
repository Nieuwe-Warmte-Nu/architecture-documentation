# OMOTES SDK

OMOTES communicated with any other application through its SDK. Current implementations:

- [Python](https://github.com/Project-OMOTES/omotes-sdk-python)
- [Typescript](https://github.com/Project-OMOTES/typescript-sdk)

The SDK communicates with OMOTES by sending and receiving AMQP 0.9 messages through 
[RabbitMQ](https://www.rabbitmq.com/). The protocol structure is defined 
[here](https://github.com/Project-OMOTES/omotes-sdk-protocol).

# Sequence diagrams

## Communication of running a job
![Example of submitting and running a job](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.github.com/Project-OMOTES/architecture-documentation/master/sdk/submit_and_run_job.puml)

## Communication of cancelling a job
![Example of cancelling a job](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.github.com/Project-OMOTES/architecture-documentation/master/sdk/cancel_job.puml)
