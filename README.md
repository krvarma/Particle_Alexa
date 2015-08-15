Particle and Amazon Echo
------------------------

Couple of weeks back I received my Amazon Echo. First thing came to mind is to try integrating with Particle (especially after reading this post http://community.particle.io/t/alexa-open-spark-and-amazon-echo-hackathon/11318/13). Here is the project I worked.

The Particle application is really simple, just read temperature and humidity from a DHT22 sensor. Also it has two functions to control two LEDs Red and Green. The firmware exposes three functions. *gettmp* returns temperature, *gethmd* returns humidity and *ctrlled* controls the LEDS attached to D2 and D6.

On the Amazon Echo side, we have to use Alexa Skill Set to interact with it. For this I have created one Alexa Skill Set. Following are some of the interactions possible:

**One-Shot Modal**
User: Alexa, ask particle, what is the temperature*
Alexa: *Temperature is 30 degree*

User: *Alexa ask particle, what is the humidity*
Alexa: *Humidity is 76%*

User: *Alexa, tell particle to turn on red light*
Alexa: *OK, red light turned on*

User: *Alexa, tell particle to turn off red light*
Alexa: *OK, red light turned off*

User: *Alexa, tell particle to turn on green light*
Alexa: *OK, green light turned on*

User: *Alexa, tell particle to turn off green light*
Alexa: *OK, green light turned off*

User: *Alexa, open particle*. 
Alexa: Alexa will open the particle and speak the help text and wait for next commands. You can ask *what is the temperature*, *what is the humidity* or tell Alexa to turn on or off lights.

To create a new Alexa Skill Set, we have to create 

1. Service that will process the voice input
2. Intent Schema that defines the actions for the voice input
2. Sample Utterance file that contains all the possible voice inputs and its corresponding intent.

Below are the sample utterance and intent schema of our application:

**Intent Schema**

    {
      "intents": [
        {
          "intent": "ParticleIntent",
          "slots": [
    	{
              "name": "sensor",
              "type": "LITERAL"
            },
            {
              "name": "light",
              "type": "LITERAL"
            },
            {
              "name": "onoff",
              "type": "LITERAL"
            }
          ]
        },
        {
          "intent": "HelpIntent",
          "slots": []
        }
      ]
    }

**Sample Utterance**

    ParticleIntent what is the {temperature|sensor} here
    ParticleIntent what is the {humidity|sensor} here
    ParticleIntent turn {on|onoff} {red|light} light
    ParticleIntent turn {on|onoff} {green|light} light
    ParticleIntent turn {off|onoff} {red|light} light
    ParticleIntent turn {off|onoff} {green|light} light
    
    ParticleIntent {temperature|sensor}
    ParticleIntent {humidity|sensor}
    ParticleIntent {red|light} 
    ParticleIntent {green|light} 
    
    HelpIntent help
    HelpIntent help me
    HelpIntent what can I ask you
    HelpIntent get help
    HelpIntent to help
    HelpIntent to help me
    HelpIntent what can you do
    HelpIntent what do you do
    HelpIntent how do I use you
    HelpIntent how can I use you
    HelpIntent what can you tell me
As you can see our application has two intents *ParticleIntent* and *HelpIntent*. *ParticleIntent* is invoked when you tell the Alexa to turn on/off lights or ask for temperature or humidity. *HelpIntent* is invoked when you ask for help.

In the intent schema, you can see *slots* section. These are parameters that should be passed to you service. The values of these parameters are defined in the utterance file. In our case these are *{temperature|sensor}*, {humidity|sensor}, etc...

When Alexa recognize these voice input, it will invoke our service application with all the slots specified if any. You can create service application in Node.js or Java and host it on your own web server or you can use the AWS Lambda functions to host. In our case I am using AWS Lambda functions and it is written in Node.js. You can see these in the GitHub Repository.

In our application we check the slots and call appropriate functions in our firmware, if the slot *sensor* is *temperature* then we call the temperature function and send the response. Echo will speak these response.

### Installation
*These installation steps are taken from one of the Alexa Skill Set sample.*

***AWS Lambda Setup***

 3. Go to the AWS Console and click on the Lambda link. Note: ensure you
    are in us-east or you won't be able to use Alexa with Lambda.
 4. Click on the Create a Lambda Function or Get Started Now button.
 4. Name the Lambda Function "Particle".
 5. Go to the the src directory, select all files and then create a zip file, make sure the zip file does not contain the src directory itself, otherwise Lambda function will not work.
 6. Upload the .zip file to the Lambda
 7. Keep the Handler as index.handler (this refers to the main js file in the zip).
 8. Create a basic execution role and click create.
 9. Return to the main Lambda page, and click on "Actions" -> "Add Event Source"
 10. Choose Alexa Skills Kit and click submit.
 11. Click on your Lambda function name and copy the ARN to be used later in the Alexa Skill Setup

***Alexa Skill Setup***

 12. Go to the Alexa Console (https://developer.amazon.com/edw/home.html) and click Add a New Skill.
 13. Set "Particle" as the skill name and "particle" as the invocation name, this is what is used to activate your skill. For example you would say: "Alexa, tell Particle to turn on red light"
 14. Select the Lambda ARN for the skill Endpoint and paste the ARN copied from above. Click Next.
 15. Copy the Intent Schema from the included IntentSchema.json.
 16. Copy the Sample Utterances from the included SampleUtterances.txt. Click Next.
 17. [optional] go back to the skill Information tab and copy the appId. Paste the appId into the index.js file for the variable APP_ID,
   then update the lambda source zip file with this change and upload to lambda again, this step makes sure the lambda function only serves request from authorized source.
 18. You are now able to start testing your sample skill! You should be able to go to the Echo webpage (http://echo.amazon.com/#skills) and see your skill enabled.
 19. In order to test it, try to say some of the Sample Utterances from the Examples section below.
 20. Your skill is now saved and once you are finished testing you can continue to publish your skill.


**Screenshots**

**Demo Video**

https://www.youtube.com/watch?v=-BkBoAMksT4