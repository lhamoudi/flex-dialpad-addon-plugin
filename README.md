# Native Flex Dialpad Add-on

The native Flex Dialpad does not support agent-to-agent direct calls or external transfers yet. This plugin is meant to be an add-on to the native Flex Diapad, adding both agent-to-agent direct calls and external transfers.

## Flex plugin

A Twilio Flex Plugin allow you to customize the appearance and behavior of [Twilio Flex](https://www.twilio.com/flex). If you want to learn more about the capabilities and how to use the API, check out our [Flex documentation](https://www.twilio.com/docs/flex).

## How it works

This plugin uses Twilio Functions and WorkerClient's createTask method to create conferences and TaskRouter tasks for orchestration in both agent-to-agent calls and external transfers features.

### Agent-to-agent direct call

This part adds a call agent section to the _Outbound Dialer Panel_. In this section, there is a dropdown where you can select the agent you want to call directly. After selecting and clicking the call button, the WorkerClient's createTask method is used to create the outbound call task having the caller agent as target. When the task is sent to this agent, the AcceptTask action is overridden so we can control all the calling process. Then, we use the reservation object inside the task payload to call the caller agent. This reservation object is part of the TaskRouter Javascript SDK bundled with Flex. The URL endpoint of this call is used and pointed to a Twilio Function that retuns a TwiML which in turns create a conference and sets the statusCallbackEvent. The latter endpoint will be used to create the called agent task.

In the called side, the AcceptTask action is also overridden and a similar calling process is done. The difference is that the URL endpoint points to a different Twilio Function that returns a simple TwiML which in turns calls the conference created on the caller side.

This feature is based on the work on this [project](https://github.com/lehel-twilio/plugin-dialpad).

### External transfer

When in a call, a "plus" icon is added to the Call Canvas where you can add a external number to the call. This action executes a Twilio Function that uses the Twilio API to make a call and add this call to the current conference. In the Flex UI side, the participant is added manually and both hold/unhold and hangup buttons are available.

This feature is based on the work on this [project](https://github.com/twilio-labs/plugin-flex-outbound-dialpad).

# Configuration

## Setup

- If you do not already have the **Twilio CLI** pleas install it in order to work with this plugin and Twilio Flex plugin deployments.
  https://www.twilio.com/docs/flex/developer/plugins/cli/install#install-flex-plugins-cli

- This plugin makes use of React 16.13.1, so make sure yo have Flex configured to use React 16.13.1 as noted here: https://www.twilio.com/docs/flex/developer/plugins/react-versions

- Make sure you have [Node.js](https://nodejs.org) as well as [`npm`](https://npmjs.com) installed.

- Afterwards, install the dependencies by running `npm install`:

```bash
cd

# If you use npm
npm install
```

## Development

In order to develop locally, you can use the Webpack Dev Server by running the following command using the **Twilio CLI**:

```bash
UNBUNDLED_REACT=true twilio flex:plugins:start
```

This will automatically start up the Webpack Dev Server and open the browser for you. Your app will run on `http://localhost:3000`. If you want to change that you can do this by setting the `PORT` environment variable:

```bash
UNBUNDLED_REACT=true twilio flex:plugins:start --port=3001
```

When you make changes to your code, the browser window will be automatically refreshed.

## Deploy

```bash
UNBUNDLED_REACT=true twilio flex:plugins:deploy
```

Your plugin is now deployged, but it is not yet live. You have two options for making a plugin live on your instance of Flex:

1. Once the plugin has been deployed it is not yet live. Please instructions found here to release the plugin using the API
   https://www.twilio.com/docs/flex/developer/plugins/cli/deploy-and-release

2. If you have Admin access to your Flex instance, then navigate to https://flex.twilio.com/admin/plugins to use the web interface to complete the plugin deployment and make it live.

Note: Common packages like `React`, `ReactDOM`, `Redux` and `ReactRedux` are not bundled with the build because they are treated as external dependencies so the plugin will depend on Flex to provide them globally.

## TaskRouter

Before using this plugin you must first create a dedicated TaskRouter workflow or just add the following filter to your current workflow. Make sure it is part of your Flex Task Assignment workspace.

- ensure the following matching worker expression: _task.targetWorker==worker.contact_uri_
- ensure the priority of the filter is set to 1000 (or at least the highest in the system)
- make sure the filter matches to a queue with Everyone on it. The default Everyone queue will work but if you want to seperate real time reporting for outbound calls, you should make a dedicated queue for it with a queue expression
  _1==1_

<img width="700px" src="screenshots/outbound-filter.png"/>

## Twilio Serverless

You will need the [Twilio CLI](https://www.twilio.com/docs/twilio-cli/quickstart) and the [serverless plugin](https://www.twilio.com/docs/labs/serverless-toolkit/getting-started) to deploy the functions inside the `serverless` folder of this project. You can install the necessary dependencies with the following commands:

`npm install twilio-cli -g`

and then

`twilio plugins:install @twilio-labs/plugin-serverless`

# How to use

1. Setup all dependencies above: the workflow and Twilio CLI packages.

2. Clone this repository

3. Copy .env.example to .env.development and to .env.production and set the following variables:

   - REACT_APP_SERVICE_BASE_URL: your Twilio Functions base url (this will be available after you deploy your functions). In local development environment, it could be your localhost base url.
   - REACT_APP_TASK_CHANNEL_SID: the voice channel SID

   **Note**: Remember that both .env.development and .env.production is for front-end use so do not add any type of key/secret variable to them. When developing, the .env.development is used while the .env.production is used when building and deploying the plugin. Also, just variables starting with the name _REACT*APP*_ will work.

4. run `npm install`

5. copy ./serverless/.env.sample to ./serverless/.env and populate the appropriate environment variables.

6. cd into ./serverless/ then run `npm install` and then `twilio serverless:deploy` (optionally you can run locally with `twilio serverless:start --ngrok=""`

7. cd back to the root folder and run `npm start` to run locally or `npm run-script build` and deploy the generated ./build/plugin-dialpad.js to [twilio assests](https://www.twilio.com/console/assets/public) to include plugin with hosted Flex

# Known issues

1. When in an agent-to-agent call, the transfer button is disabled.
2. When in an agent-to-agent call, an external transfer is done correctly but the UI does not reflect what is going on.

# Old issues

**Note**: If you are suffering from any of the following issues, please update your plugin with the last version of this repository.

1. In the first versions, the environment variables were set by the UI Configuration (please refer to this [documentation](https://www.twilio.com/docs/flex/ui/configuration)) but it was overriding some other variables with no relation to this plugin. Because of that, some features inside Flex were breaking. Now, there are two files (.env.development and .env.production) that gather all the environment variables.
2. Before, the worker's contact_uri was extracted from `manager.user.identity` which has its problems depending on its format. It is now being extract from `manager.workerClient.attributes.contact_url` directly. (Thanks to [@hgs-berlee](https://github.com/hgs-berlee) who pointed that out and suggested this solution)
3. Before, when in an external transfer, the hold/unhold button was executing these actions on the first participant and not on the correct one. Now, this is fixed.
