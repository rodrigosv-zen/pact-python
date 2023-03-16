# Examples

## Table of Contents

  * [Overview](#overview)
  * [broker](#broker)
  * [common](#common)
  * [consumer](#consumer)
  * [flask_provider](#flask_provider)
  * [pacts](#pacts)

## Overview

Here you can find examples of how to use Pact using the python language. You can find more of an overview on Pact in the
[Pact Introduction].

Examples are given of both the [Consumer] and [Provider], this does not mean however that you must use python for both.
Different languages can be mixed and matched as required.

In these examples, `1` is just used to meet the need of having *some* [Consumer] or [Provider] version. In reality, you
will generally want to use something more complicated and automated. Guidelines and best practices are available in the
[Versioning in the Pact Broker]

## broker

The [Pact Broker] stores [Pact file]s and [Pact verification] results. It is used here for the [consumer](#consumer),
[flask_provider](#flask-provider) and [message](#message) tests.

### Running

These examples run the [Pact Broker] as part of the tests when specified. It can be run outside the tests as well by
performing the following commands from a separate terminal:

Start postgres
```bash
docker run -d --name postgres -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgres -e POSTGRES_DB=postgres -p 5432:5432 postgres
```
Start the broker
```bash
docker run --name pact-broker --link postgres:postgres -e PACT_BROKER_DATABASE_USERNAME=postgres -e PACT_BROKER_DATABASE_PASSWORD=password -e PACT_BROKER_DATABASE_HOST=192.168.64.9 -e PACT_BROKER_DATABASE_NAME=postgres -e PACT_BROKER_BASIC_AUTH_USERNAME=pactbroker -e  PACT_BROKER_BASIC_AUTH_PASSWORD=pactbroker -p 80:9292 pactfoundation/pact-broker
```
if the database host doesn't match, run:
```bash
minikube ip
```
and replace the result in the ```PACT_BROKER_DATABASE_HOST``` above, and in the next places:

  * examples/common/sharedfixtures.py ```PACT_BROKER_BASE_URL```
  * examples/consumer/tests/consumer/test_user_consumer.py ```PACT_BROKER_URL```
  * examples/flask_provider/verify_pact.sh ```--pact-url```

The broker takes some time to set up the app to release the UI, after that, you should then be able to open a browser and navigate to http://192.168.64.9, it will open a form to log in, just use ```pactbroker``` as username and password, then you will be able to see the default Example App/Example API Pact.

Running the [Pact Broker] outside the tests will mean you are able to then see the [Pact file]s submitted to the
[Pact Broker] as the various tests are performed.

## common

To avoid needing to duplicate certain fixtures, such as starting up a docker based Pact broker (to demonstrate how the
test process could work), the shared fixtures used by the pytests have all been placed into a single location.]
This means it is easier to see the relevant code for the example without having to go through the boilerplate fixtures.
See [Requiring/Loading plugins in a test module or conftest file] for further details of this approach.

## consumer

Pact is consumer-driven, which means first the contracts are created. These Pact contracts are generated during
execution of the consumer tests.

### Running

When the tests are run, the "minimum" is to generate the Pact contract JSON, additional options are available. The
following commands can be run from the `examples/consumer` folder:
- Create a virtual environment:
    ```bash
    python -m venv venv
    ```
- Activate the virtual environment:
    ```bash
    source venv/bin/activate
    ```
- Install any necessary dependencies:
    ```bash
    pip install -r requirements.txt
    ```
- To run the tests, and publish the results to the broker which is already running:
    ```bash
    pytest --publish-pact 1
    ```
- To just run the tests:
    ```bash
    pytest
    ```

### Output

The following file(s) will be created when the tests are run:

| Filename                                                   | Contents                                                 |
|------------------------------------------------------------| ---------------------------------------------------------|
| consumer/pact-mock-service.log                             | All interactions with the mock provider such as expected interactions, requests, and interaction verifications. |
| consumer/tests/consumer/userserviceclient-userservice.json | This contains the Pact interactions between the `UserServiceClient` and `UserService`, as defined in the tests. The naming being derived from the named Pacticipants: `Consumer("UserServiceClient")` and `Provider("UserService")` |

## flask_provider
examples/consumer/tests/consumer/userserviceclient-userservice.json
The Flask [Provider] example consists of a basic Flask app, with a single endpoint route.
This implements the service expected by the [consumer](#consumer).

Functionally, this provides the same service and tests as the [fastapi_provider](#fastapi_provider). Both are included to
demonstrate how Pact can be used in different environments with different technology stacks and approaches.

The [Provider] side is responsible for performing the tests to verify if it is compliant with the [Pact file] contracts
associated with it.

As such, the tests use the pact-python Verifier to perform this verification. Two approaches are demonstrated:
- Testing against the [Pact broker]. Generally this is the preferred approach, see information on [Sharing Pacts].
- Testing against the [Pact file] directly. If no [Pact broker] is available you can verify against a static [Pact file].

### Running

To avoid package version conflicts with different applications, it is recommended to run these tests from a separated terminal and a new
[Virtual Environment]

The following commands can be run from the `examples/flask_provider`:
- Create a virtual environment:
    ```bash
    python -m venv venv
    ```
- Activate the virtual environment:
    ```bash
    source venv/bin/activate
    ```
- Install any necessary dependencies:
    ```bash
    pip install -r requirements.txt
    pip install -e ../../           # Using setup.py in the pact-python root, install any pact dependencies and pact-python
    ```

To perform verification using CLI to verify the [Pact file] against the Flask [Provider] instead of the python tests:
```bash
./verify_pact.sh                # Wrapper script to first run Flask, and then use `pact-verifier` to verify locally
```

To perform verification using CLI, but verifying the [Pact file] previously provided by a [Consumer], and publish the
results. This example requires that the [Pact broker] is already running, and the [Consumer] tests have been published
already, described in the [consumer](#consumer) section above.
```bash
./verify_pact.sh 1              # Wrapper script to first run Flask, and then use `pact-verifier` to verify and publish
```

These examples demonstrate by first launching Flask via a `python -m flask run`, you may prefer to start Flask using an
`app.run()` call in the python code instead, see [How to Run a Flask Application]. Additionally for tests, you may want
to manage starting and stopping Flask as part of a fixture setup. Any approach can be chosen here, in line with your
existing Flask testing practices.

### Output

The following file(s) will be created when the tests are run

| Filename                    | Contents  |
|-----------------------------| ----------|
| flask_provider/log/pact.log | All Pact interactions with the Flask Provider. Every interaction example retrieved from the Pact Broker will be performed during the Verification test; the request/response logged here. | 

## pacts

The Flask [Provider] example implement the service the [Consumer] example interacts with.
This folder contains the generated [Pact file] for reference, which is also used when running the [Provider] tests
without a [Pact Broker].

[Pact Broker]: https://docs.pact.io/pact_broker
[Pact Introduction]: https://docs.pact.io/
[Consumer]: https://docs.pact.io/getting_started/terminology#service-consumer
[Provider]: https://docs.pact.io/getting_started/terminology#service-provider
[Versioning in the Pact Broker]: https://docs.pact.io/getting_started/versioning_in_the_pact_broker/
[Pact file]: https://docs.pact.io/getting_started/terminology#pact-file
[Pact verification]: https://docs.pact.io/getting_started/terminology#pact-verification]
[Virtual Environment]: https://docs.python.org/3/tutorial/venv.html
[Sharing Pacts]: https://docs.pact.io/getting_started/sharing_pacts/]
[How to Run a Flask Application]: https://www.twilio.com/blog/how-run-flask-application
[Requiring/Loading plugins in a test module or conftest file]: https://docs.pytest.org/en/6.2.x/writing_plugins.html#requiring-loading-plugins-in-a-test-module-or-conftest-file