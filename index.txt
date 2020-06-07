'strict mode';



var http = require('http');
var cow = 0;


exports.handler = function(event, context) {
    try {
        // Good to initalize attributes
         var request = event.request;
         var session = event.session;
        if(!event.session.attributes) {
          event.session.attributes = {};
        }
        
        let options = {};
        
         if (request.type === "LaunchRequest") {
           handleLaunchRequest(context);
    
         } else if (request.type === "IntentRequest") {
           if (request.intent.name === "ActionIntent") {
             cow = 0;
             ChecklistIntentChecker(request, context);
           } else if(request.intent.name === "ChecklistIntentChecker") {
             ChecklistIntentChecker(request,context,session);
           } else if(request.intent.name === "AMAZON.YesIntent") {
              let options = {};
              cow += 1;
              if(typ === "inside") {
                options.speechText = checklist1[cow];
              } else {
                options.speechText = checklist2[cow];
              }
              options.endSession = false;
              context.succeed(buildResponse(options));
           } else if(request.intent.name === "AMAZON.NoIntent") {
              let options = {};
              if(typ === "inside" && (cow == 4 || cow == 5 )) {
                options.speechText = "Hey, you should tell your guardian that, <break time=\".2s\"/> don't worry I'll wait for you, just let me know once you tell them.";
                options.endSession = false;
                context.succeed(buildResponse(options));
              } else {
                options.speechText = "Whoops<break time=\".1s\"/> you might want to do that, don't worry I'll wait for you, just let me know once you are done. <break time=\"10s\"/>";
                options.endSession = false;
                context.succeed(buildResponse(options));
              }
           } else if(request.intent.name === "AMAZON.HelpIntent") {
             let options = {};
             options.speechText = "Hey, just say what you would like help with. You can say the checklist?";
             options.endSession = false;
             context.succeed(buildResponse(options));
           } else if(request.intent.name === "AMAZON.FallbackIntent") {
             let options = {};
             options.speechText = "Oh no looks like something went wrong, try again by saying an action.";
             options.endSession = false;
             context.succeed(buildResponse(options));
           } else if(request.intent.name === "ThanksIntent") {
             context.succeed(buildResponse({
               speechText: "Good bye,and remember to be safe!",
               endSession: true
             }));
           }  else {
              throw "Unknown Intent";
           }
    
          } else if (request.type === "SessionEndedRequest") {
    
          } else {
             throw "Unknown intent type";
          }
           
       } catch(e) {
         context.fail("Excpetion: " + e);
       }
};



function buildResponse(options) {
  var response = {
    version: 1.0,
    response: {
      outputSpeech: {
        type: "SSML",
        ssml: "<speak>" + options.speechText + "</speak>"
      },
      shouldEndSession: options.endSession
    }
  };

  if(options.repromptText) {
    response.response.reprompt = {
      outputSpeech: {
        type: "SSML",
        ssml: "<speak>" + options.repromptText + "</speak>"
      }
    };
  }
  
  if(options.session && options.session.attributes) {
    response.sessionAttributes = options.session.attributes;
  }
  
  return response;
  
}

function handleLaunchRequest(context) {
  let options = {};
           options.speechText = "Hey there! Are you getting ready to leave or did you enter?";
           options.repromptText = " You can say for example, I'm getting ready to leave.";
           options.endSession = false;
           context.succeed(buildResponse(options));
}

  
function randomQuotes(arr){  
    var factIndex = Math.floor(Math.random() * arr.length);
    var randomQuote = arr[factIndex];
    return randomQuote;
 }


function handleHelloIntent(request, context) {
  let options = {};
  var name = request.intent.slots.cow.value; 
  if(name.indexOf("list") != -1 ) {
    options.speechText = "Nice, are you getting ready to leave or did you just enter?";
    options.endSession = false;
    context.succeed(buildResponse(options));
  } else if (name.indexOf("symptoms") != -1) {
    options.speechText += "Thats unfortunate, let me list off the symptoms so that you can compare!";
    options.endSession = false;
    context.succeed(buildResponse(options));
  } else {
    options.speechText = "Oh no looks like something went wrong<break time=\".1s\"/>, restart the skill by asking alexa to open it again.";
    options.endSession = true;
    context.succeed(buildResponse(options));
  }
  
}


var checklist1 = [
    "Did you remove your mask and gloves?",
    "Did you change your clothes if they got dirty?",
    "Did you wash your hands for twenty seconds with soap and water?",
    "Did you clean any items that you might have bought?",
    "Did you make sure to not touch your eyes and mouth?!",
    "Did you stay away from anyone who might have been coughing.",
    "You finished the checklist, you're all good to go now!"
  ];
  
var checklist2 = [
    "Did you remember to wear your mask and gloves?",
    "Do you have your keys if you need them?",
    "Do you have hand sanitizer?",
    "Thats all!, just remember to keep your hands away from your face and to stay practice social distancing!"
  ];
  
var typ = "";
function ChecklistIntentChecker(request, context, session) {
  let options = {};
  var cows = request.intent.slots.cow.value; 
  if(cows.indexOf("inside") != -1 || cows.indexOf("ent") != -1 || cows.indexOf("in") != -1 || cows.indexOf("coming") != -1 || cows.indexOf("came") != -1) {
    options.speechText = checklist1[cow];
    typ = "inside";
    if(cow == 6) {
      options.endSession= true;
      context.succeed(buildResponse(options));
    } else {
      options.endSession = false;
      cow = 0;
      context.succeed(buildResponse(options));
    }
  } else if (cows.indexOf("leave") != -1 || cows.indexOf("lea") != -1 || cows.indexOf("go") != -1 || cows.indexOf("le") != -1 || cows.indexOf("am ready to leave") != -1 || cows.indexOf("am") != -1) {
    options.speechText = checklist2[cow];
    typ = "outside";
    if(cow == 3) {
      options.endSession= true;
      context.succeed(buildResponse(options));
    } else {
      options.endSession = false;
      context.succeed(buildResponse(options));
    }
  } else {
    options.speechText = "Looks like something went wrong, say wether you are getting ready or just arrived.";
    options.endSession = false;
    context.succeed(buildResponse(options));
}

}

