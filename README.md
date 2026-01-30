##React
# How load the environment variables from .env.build?
- dockerfile
  - pnpm build # Build command
- package.json
  - "build": "cross-env PORT=4006 NODE_ENV=production SERVER=prod webpack --config config/webpack/webpack.prod.js --color --mode production --progress",
   - dotenv-webpack": "^8.0.1" # Using this plugin to explictly load .env.build file
- config/webpack/webpack.prod.js
        const Dotenv = require('dotenv-webpack');
        const plugins = [
          new Dotenv({
            path: '.env.build',
            systemvars: true,
          })
       ];

# How to update the value of envirment variable with respect to environment? In case of use the same image on all environments
   - using sed command
   - .env.build
         - REMOTE_ENTRY_NAME="remoteentry.js"  --> common to all envs and no need of sed replacement
         - API_URL="API_URL_VALUE" --> specific value for each environment
   - Dockerfile 
      Copy start script
      COPY start.sh /start.sh
      RUN chmod +x /start.sh
      EXPOSE 80
     CMD ["/start.sh"]
   - start.sh
     
    Print log for debugging purposes
    echo "Replacing environment variables in /usr/share/nginx/html/ ..."
    cd /usr/share/nginx/html/
    
    Below commands replace placeholders like API_URL_VALUE in static files during deployment.
    Please add one line like below for every environment variable for the replacement during deployment.
    sed -i "s@API_URL_VALUE@$API_URL@g"  *
    
    Start nginx in the foreground
    nginx -g 'daemon off;'
  - configmap
      - we get the API_URL_VALUE from POD system environment variable which we set using configmap.



# How to validate
  - add console.log in the code like console.log(process.env.ENV_VAR_NAME);
  - open pod and goto /usr/share/nginx/html and lookk for 572 chunk JS file. Copy to your local machine and check the variable/values.
      - kubectl cp POD_NAME:/usr/share/nginx/html/572.XXXXXXXXXXXXXXXXXXXX.chunk.js  chunk.js -n NAMESPACE # XXX - Replace with actual filename.
 
