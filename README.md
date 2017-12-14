var housemates = {
    "heather" : {
        "personality_trait": "is incredibly hilarious",
        "skill": "enjoys running"
    }, 
    "rob" : {
        "personality_trait": "tends to be pessimistically existential",
        "skill": "loves to eat ass"
    }, 
    "jerome" : {
        "personality_trait": "is a biological and chemical encyclopedia",
        "skill": "loves to cook"
    }, 
    "reese" : {
        "personality_trait": "enjoys learning new skills",
        "skill": "making margaritas"
    }, 
    "jacqueline" : {
        "personality_trait" : "tends to disappear into the weekend",
        "skill" : "tells nice stories that have a nice idea but then like go on tangents sometimes but then it sometimes gets interesting"
    }
}


// Route the incoming request based on type (LaunchRequest, IntentRequest,
// etc.) The JSON body of the request is provided in the event parameter.
exports.handler = function (event, context) {
    try {
        console.log("event.session.application.applicationId=" + event.session.application.applicationId);

        /**
         * Uncomment this if statement and populate with your skill's application ID to
         * prevent someone else from configuring a skill that sends requests to this function.
         */

    if (event.session.application.applicationId !== "amzn1.ask.skill.34d4d04e-077d-48e0-a2f2-ae0313490d61") {
        context.fail("Invalid Application ID");
     }

        if (event.session.new) {
            onSessionStarted({requestId: event.request.requestId}, event.session);
        }

        if (event.request.type === "LaunchRequest") {
            onLaunch(event.request,
                event.session,
                function callback(sessionAttributes, speechletResponse) {
                    context.succeed(buildResponse(sessionAttributes, speechletResponse));
                });
        } else if (event.request.type === "IntentRequest") {
            onIntent(event.request,
                event.session,
                function callback(sessionAttributes, speechletResponse) {
                    context.succeed(buildResponse(sessionAttributes, speechletResponse));
                });
        } else if (event.request.type === "SessionEndedRequest") {
            onSessionEnded(event.request, event.session);
            context.succeed();
        }
    } catch (e) {
        context.fail("Exception: " + e);
    }
};

/**
 * Called when the session starts.
 */
function onSessionStarted(sessionStartedRequest, session) {
    // add any session init logic here
}

/**
 * Called when the user invokes the skill without specifying what they want.
 */
function onLaunch(launchRequest, session, callback) {
    getWelcomeResponse(callback)
}

/**
 * Called when the user specifies an intent for this skill.
 */
function onIntent(intentRequest, session, callback) {

    var intent = intentRequest.intent
    var intentName = intentRequest.intent.name;

    // dispatch custom intents to handlers here
    if (intentName == "ResidentsIntent") {
        handleResidentsIntent(intent, session, callback)
    } else if (intentName == "AMAZON.YesIntent") {
        handleYesResponse(intent, session, callback)
    } else if (intentName == "AMAZON.NoIntent") {
        handleNoResponse(intent, session, callback)
    } else if (intentName == "AMAZON.HelpIntent") {
        handleGetHelpRequest(intent, session, callback)
    } else if (intentName == "AMAZON.StopIntent") {
        handleFinishSessionRequest(intent, session, callback)
    } else if (intentName == "AMAZON.CancelIntent") {
        handleFinishSessionRequest(intent, session, callback)
    } else {
        throw "Invalid intent"
    }
}

/**
 * Called when the user ends the session.
 * Is not called when the skill returns shouldEndSession=true.
 */
function onSessionEnded(sessionEndedRequest, session) {

}

// ------- Skill specific logic -------

function getWelcomeResponse(callback) {
    var speechOutput = "Welcome to 6524 Calle Koral! Ask me about the wonderful humans in this house: " + "Heather, Rob, Jerome, Reese, and Jacqueline." +
    "I can only discuss one person at a time. Who would you like to start with?"

    var reprompt = "Hello? Who would you like to know more about?"

    var header = "The House of Calle Koral"

    var shouldEndSession = false

    var sessionAttributes = {
        "speechOutput" : speechOutput,
        "repromptText" : reprompt
    }

    callback(sessionAttributes, buildSpeechletResponse(header, speechOutput, reprompt, shouldEndSession))

}

function handleResidentsIntent(intent, session, callback) {
    var housemate = intent.slots.Housemates.value.toLowerCase()

    if (!housemates[housemate]) {
        var speechOutput = "Sorry, that person does not exist in the house. Please try someone that lives here."
        var repromptText = "Try asking about someone that lives here."
        var header = "Not Important To Us"
    } else {
        var personality_trait = housemates[housemate].personality_trait
        var skill = housemates[housemate].skill
        var speechOutput = capitalizeFirst(housemate) + " " + personality_trait + " and " + skill + ". Do you want to hear about more housemates?"    
        var repromptText = "Do you want to hear about more housemates"
        var header = capitalizeFirst(housemate)
    }

    var shouldEndSession = false

    callback(session.attributes, buildSpeechletResponse(header, speechOutput, repromptText, shouldEndSession))
}

function handleYesResponse(intent, session, callback) {
    var speechOutput = "Dope! Who would you like to learn more about?"
    var repromptText = speechOutput
    var shouldEndSession = false

    callback(session.attributes, buildSpeechletResponseWithoutCard(speechOutput, repromptText, shouldEndSession))
}

function handleNoResponse(intent, session, callback) {
    handleFinishSessionRequest(intent, session, callback)
}

function handleGetHelpRequest(intent, session, callback) {
    // Ensure that session.attributes has been initialized
    if (!session.attributes) {
        session.attributes = {};
    }

    var speechOutput = "I can tell you about all our wonderful housemates: " + "Heather, Rob, Jerome, Reese, and Jacqueline. " + "Who would you like to learn about?"

    var repromptText = speechOutput

    var shouldEndSession = false

    callback(session.attributes, buildSpeechletResponseWithoutCard(speechOutput, repromptText, shouldEndSession))

}

function handleFinishSessionRequest(intent, session, callback) {
    // End the session with a "Good bye!" if the user wants to quit the game
    callback(session.attributes,
        buildSpeechletResponseWithoutCard("Thank you for learning about our wonderful housemates! Until next time.", "", true));
}


// ------- Helper functions to build responses for Alexa -------


function buildSpeechletResponse(title, output, repromptText, shouldEndSession) {
    return {
        outputSpeech: {
            type: "PlainText",
            text: output
        },
        card: {
            type: "Simple",
            title: title,
            content: output
        },
        reprompt: {
            outputSpeech: {
                type: "PlainText",
                text: repromptText
            }
        },
        shouldEndSession: shouldEndSession
    };
}

function buildSpeechletResponseWithoutCard(output, repromptText, shouldEndSession) {
    return {
        outputSpeech: {
            type: "PlainText",
            text: output
        },
        reprompt: {
            outputSpeech: {
                type: "PlainText",
                text: repromptText
            }
        },
        shouldEndSession: shouldEndSession
    };
}

function buildResponse(sessionAttributes, speechletResponse) {
    return {
        version: "1.0",
        sessionAttributes: sessionAttributes,
        response: speechletResponse
    };
}

function capitalizeFirst(s) {
    return s.charAt(0).toUpperCase() + s.slice(1)
}
