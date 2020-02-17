# Socrates BE Serverless framework

## Set up

Goal: Create a simple lambda that can be accessed through an API gateway. 
Deploy this code using Amazon Codebuild and the Serverless framework.

## Steps
1. In the AWS console
    1. Create a lambda (nodeJS 12.x)
    1. Paste in this code:
      ```javascript
      exports.listPets = async (event) => {
          const response = {
              statusCode: 200,
              body: JSON.stringify(
              {
              message: 'getPet API',
              names: ['Tom','Jerry','Spike'],
              date: new Date()
              }
                  ),
          };
          return response;
      };
      ```
    1. Add an API Gateway as a trigger (HTTP API)
    1. Save it all
    1. Call the endpoint to verify it works.
1. Use serverless for deploys.
    1. On local machine: `npm init`
    1. Copy the lambda code to `petstore.js`
    1. Install serverless: `npm install serverless`
    1. Create the `serverless.yml` file
       ```yaml
       service: PetStore
       
       provider:
         name: aws
         runtime: nodejs12.x
         region: eu-central-1
       
       functions:
         listPet:
           handler: petstore.listPets
           events:
             - http:
                 path: /pet
                 method: get
       ```
    1. Before the first time, execute the `serverless config credentials` script. 
       It needs AWS credentials (Access key and secret) that have all required roles.
       Serverless deploys will fail and mention the missing roles in the error message.
    1. Run `/node_modules/serverless/bin/serverless deploy` (unless you installed serverless globally.)
    1. Notice that the petstore.zip is 26MB in size.
       This is because the serverless framework is included in the .zip file.
       Move that to the `devDependencies` in the `package.json` file and see the size drop to less than 50kB.
    1. The deploy command will show you the URL where the API is available.
       Verify that it works ;)
1. Set up git repo (socratesbe-mob -> petstore)
    1. Add all required files (do *not* push node_modules, package-lock.json or .serverless/)
    1. Push it all.
1. Create AWS CodePipeline for automatic builds/deploys.
    1. Use the correct role ARN (`AWSCodePipelineServiceRole-eu-west-2-getpet-pipeline`) as this already has all the required roles.
    1. Set source provider to github. 
    1. Authorize the application (let Davy enter the password).
    1. Select the petStore repo (master branch).
    1. Provider: CodeBuild
    1. Region (EU Paris)
    1. Click the `Create Project` button
    1. Give it a name
    1. Use the Ubuntu managed image
    1. Keep everything default *except* the service role.
    1. Use an existing service role: `codeBuild-getPet-service- role`
    1. Before continuing, you need to create the `buildspec.yml` file in the root of the project and push it to GitHub.
       ```yaml
        version: 0.2
        
        phases:
          install:
            runtime-versions:
                nodejs: 12
          pre_build:
            commands:
              - npm install --no-progress --silent -g serverless
              - npm --version
          build:
            commands:
              - npm run test
              - serverless -v deploy
        ```
    1. Skip the deploy stage
    1. Create pipeline
    1. Pray
