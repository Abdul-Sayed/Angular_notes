# Angular Deployment

For Front End projects, its best to use a static website host such as Firebase, netlify, vercel, and AWS S3

For serverside code, use something like Heroku

## Ahead of time Compolation (AOT) vs Just in time Compilation (JIT)

By default, during development angular uses JIT which helps with debugging. It contains the heavy Angular compiler which compiles the template code into JS DOM instructions. Before deployment, this compilation must already be done, and the compiled code, without the angular compiler included in the production bundle. This is AOT. Also, all the TS code mmust be transpiled into JS.

This can be done with `ng build --prod`

Use environment variables for any apiKeys (add them into the env files, then import environment in the files using the apiKey and access the apiKey from there).

Run ng build --prod

Next the build artifacts in the dist folder must be sent to a static host. For Firebase host:

        npm install -g firebase-tools
        firebase --version
        firebase login
        firebase projects:list
        firebase init
            -> Hosting: Configure files for firebase hosting
            -> Use an existing project
            -> Set up the workflow to run a build script before every deploy? Y
            -> What script should be run before every deploy? npm ci && npm run build
            -> ? Set up automatic deployment to your site's live channel when a PR is merged? Yes
            -> ? What is the name of the GitHub branch associated with your site's live channel? main
        firebase deploy

    Now whenever you add, commit, and push code to main, the live firebase hosted site will automatically have the changes
