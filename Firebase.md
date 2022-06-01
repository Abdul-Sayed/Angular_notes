# Angular Firebase setup

Sign in with a google account

Go to console

Create a project

On left click Build, Realtime Database, start in test mode

Cope to URL of the Realtime Database, and use for http calls

In Relatime Database, in Rules, set auth != null, and publish changes

{
"rules": {
".read": "auth != null",
".write": "auth != null",
}
}

On left, click Authentication

Choose Email/Password, enable and save

Check other auth methods by googling: firebase auth rest api

On left, click Project Overview, and copy the Web Api Key

paste it into all environment files as:

export const environment = {
production: false,
apiKey: 'your_web_api_key',
};
