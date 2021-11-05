# Dapr Quickstart

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec tempor sapien nec purus volutpat aliquam. Vivamus pretium dapibus consequat. Vestibulum tempus magna mauris, mattis rutrum elit vehicula ac. Praesent posuere, lectus sed fermentum faucibus, nibh augue lobortis ipsum, at auctor nulla ligula et nisi. Nulla sapien ligula, sagittis id justo sed, laoreet volutpat mi. Morbi ac efficitur libero, ut pulvinar leo. Fusce accumsan ultrices diam, eget viverra ligula bibendum egestas.

* **Date:** 8th November 2021
* **Squad:** Cloud Native
* **Duration:** 30 minutes

## Getting Started

For this demo we will be using the Dapr project's [quickstarts](https://github.com/dapr/quickstarts) repo. We'll focus on the [pub-sub](https://github.com/dapr/quickstarts/tree/master/pub-sub) quickstart, which demonstrates how to use Dapr to enable pub-sub applications using Redis as a pub-sub component.

1. In a new tab, navigate to https://github.com/dapr/quickstarts.

1. If you would like to jump in right away by using GitHub Codespaces, you can click on "Code" in this repo, select "Codespaces" and click "New Codespace", which will let you get started right away with ```dapr init```.

    > GitHub Codespaces will provide you a Visual Studio Code based editor backed by high performance VMs in the cloud. It will take a moment for GitHub to create your dev space. This does the following:
    >
    > - Build a container image based on the [Dockerfile](https://github.com/dapr/quickstarts/blob/v1.4.0/.devcontainer/Dockerfile). 
    > - Create a dev container based on [devcontainer.json](https://github.com/dapr/quickstarts/blob/v1.4.0/.devcontainer/devcontainer.json).
    > - Configure this container, and your dev environment as per the provided config. 
    > If you don't have access to GitHub Codespaces, follow the guide here to get started using [Dev Containers](https://code.visualstudio.com/docs/remote/containers) locally.  

1. Once your Codespace is ready. Open the Terminal and run ```dapr init```. Dapr runs as a sidecar alongside your application, and in self-hosted mode this means it is a process on your local machine. Therefore, initializing Dapr includes fetching the Dapr sidecar binaries and installing them locally. Find out more [here](https://docs.dapr.io/getting-started/install-dapr-selfhost/).

## Exploring Pub-Sub

In this quickstart, you'll create a publisher microservice and two subscriber microservices to demonstrate how Dapr enables a publish-subcribe pattern. The publisher will generate messages of a specific topic, while subscribers will listen for messages of specific topics.

Visit [this](https://docs.dapr.io/developing-applications/building-blocks/pubsub/) link for more information about Dapr and Pub-Sub.

This quickstart includes one publisher:

- React front-end message generator

And two subscribers: 
 
- Node.js subscriber
- Python subscriber

Dapr uses pluggable message buses to enable pub-sub, and delivers messages to subscribers in a [Cloud Events](https://github.com/cloudevents/spec) compliant message envelope. in this case you'll use Redis Streams. The following architecture diagram illustrates how components interconnect locally:

![Architecture Diagram](./img/Local_Architecture_Diagram.png)

### Running Locally

In order to run the pub/sub quickstart locally, each of the microservices need to run with Dapr. Start by running message subscribers. 

#### Run Node message subscriber with Dapr

1. Navigate to Node subscriber directory in your CLI:

    ```bash
    cd pub-sub
    cd node-subscriber
    ```

1. Install dependencies: 

    ```bash
    npm install
    ```

1. Run the Node subscriber app with Dapr: 

    ```bash
    dapr run --app-id node-subscriber --app-port 3000 node app.js
    ```

    > `app-id` which can be any unique identifier for the microservice. `app-port`, is the port that the Node application is running on. Finally, the command to run the app `node app.js` is passed last.

#### Run Python message subscriber with Dapr

1. Open a new CLI window and navigate to Python subscriber directory in your CLI: 

    ```bash
    cd pub-sub/python-subscriber
    ```

1. Install dependencies: 

    ```bash
    pip3 install -r requirements.txt 
    ```

1. Run the Python subscriber app with Dapr: 

    ```bash
    dapr run --app-id python-subscriber --app-port 5000 python3 app.py
    ```

#### Run the React front end with Dapr

Now, run the React front end with Dapr. The front end will publish different kinds of messages that subscribers will pick up.

1. Open a new CLI window and navigate to the react-form directory:

    ```bash
    cd pub-sub/react-form
    ```

1. Run the React front end app with Dapr: 

    ```bash
    npm run buildclient
    npm install
    dapr run --app-id react-form --app-port 8080 npm run start
    ```

    > This may take a minute, as it downloads dependencies and creates an optimized production build. You'll know that it's done when you see `== APP == Listening on port 8080!` and several Dapr logs.

1. GitHub Codespaces uses port forwarding to give you access to TCP ports running within your codespace. In your codespace, under the text editor, click "Ports".

1. Under the list of ports, find "8080" and open the "Local Address" in a new tab. You should see a form with a dropdown for message type and message text: 

1. Pick a topic, enter some text and fire off a message! Observe the logs coming through your respective Dapr subscriber.

> Note that the Node.js subscriber receives messages of type "A" and "B", while the Python subscriber receives messages of type "A" and "C". Note that logs are showing up in the console window where you ran each one: 
> ```bash
> == APP == Listening on port 8080!
> ```

#### Use the CLI to publish messages to subscribers

The Dapr CLI provides a mechanism to publish messages for testing purposes.

1. Open a new CLI window, use Dapr CLI to publish a message:

    ```bash
    cd pub-sub
    dapr publish --publish-app-id react-form --pubsub pubsub --topic A --data-file message_a.json
    ```

1. **Optional:** Try publishing a message of topic B. You'll notice that only the Node app will receive this message. The same is true for topic 'C' and the python app.

    > **Note:** If you are running in an environment without easy access to a web browser, the following curl commands will simulate a browser request to the node server.

    ```bash
    curl -s http://localhost:8080/publish -H Content-Type:application/json --data @message_b.json
    curl -s http://localhost:8080/publish -H Content-Type:application/json --data @message_c.json
    ```

1. **Optional:** You can also use Dapr's service invocation API to publish messages:

    ```bash
    curl -s http://localhost:36263/v1.0/invoke/react-form/method/publish -H Content-Type:application/json --data @message_a.json
    curl -s http://localhost:36263/v1.0/invoke/react-form/method/publish -H Content-Type:application/json --data @message_b.json
    curl -s http://localhost:36263/v1.0/invoke/react-form/method/publish -H Content-Type:application/json --data @message_c.json
    ```

#### Cleanup

1. Get Dapr to stop all the running application:

    ```bash
    dapr stop --app-id node-subscriber
    dapr stop --app-id python-subscriber
    dapr stop --app-id react-form
    ```

## Acknowledgements

* This demo is based on Dapr's [Pub-Sub quickstart](https://github.com/dapr/quickstarts/tree/v1.4.0/pub-sub).

## Contributors

* Kevin Harris - kevin.harris@microsoft.com
* Liam Moat - liam.moat@microsoft.com
* Anu Bhattacharya - anulekha.bhattacharya@microsoft.com
* Gordon Byers - gobyers@microsoft.com