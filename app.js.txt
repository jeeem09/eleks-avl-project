// --- ⚠️ CONFIGURATION - EDIT THIS! ---
const mqttConfig = {
    host: "a223565bd34942d2866e8be775b11451.s1.eu.hivemq.cloud", // e.g., "abcdef123.s1.hivemq.cloud"
    port: 8884, // Use port 8884 for secure WebSockets
    username: "janhez_motovlog",
    password: "Eleks123456789",
    clientId: "web-ui-" + Math.random().toString(16).substr(2, 8) // Random client ID
};
// -------------------------------------

// MQTT Topic Names
const LIGHTS_TOPIC = "home/lights/set";
const VIDEO_TOPIC = "home/video/set";
const AUDIO_TOPIC = "home/audio/set";

// 1. Create MQTT Client
const client = new Paho.MQTT.Client(mqttConfig.host, mqttConfig.port, mqttConfig.clientId);

// 2. Set Connection Callbacks
client.onConnectionLost = function (responseObject) {
    if (responseObject.errorCode !== 0) {
        console.log("onConnectionLost:" + responseObject.errorMessage);
    }
};

client.onMessageArrived = function (message) {
    console.log("Message Arrived: " + message.payloadString);
};

// 3. Connect to Broker
client.connect({
    onSuccess: onConnect,
    onFailure: onFailure,
    userName: mqttConfig.username,
    password: mqttConfig.password,
    useSSL: true // IMPORTANT: Must be true for port 8884
});

function onFailure(err) {
    console.error("Failed to connect: " + err.errorMessage);
    alert("Failed to connect to the Smart Home system. Check console.");
}

function onConnect() {
    console.log("Connected to MQTT Broker!");
}

// 4. Helper function to publish a message
function publish(topic, payload) {
    if (!client.isConnected()) {
        console.error("Not connected!");
        alert("Not connected to system. Please refresh.");
        return;
    }
    const message = new Paho.MQTT.Message(payload);
    message.destinationName = topic;
    client.send(message);
    console.log(`Published to ${topic}: ${payload}`);
}

// 5. Add Button Click Handlers
document.getElementById("movieBtn").onclick = function() {
    console.log("Movie Mode Activated");
    // Lights: Dim, blue color
    publish(LIGHTS_TOPIC, '{"color":"blue", "brightness":50}');
    // Video: Turn TV on
    publish(VIDEO_TOPIC, "power_on");
    // Audio: Stop music
    publish(AUDIO_TOPIC, "stop");
};

document.getElementById("studyBtn").onclick = function() {
    console.log("Study Mode Activated");
    // Lights: Bright, white color
    publish(LIGHTS_TOPIC, '{"color":"white", "brightness":200}');
    // Video: Turn TV off
    publish(VIDEO_TOPIC, "power_off");
    // Audio: Play quiet study music (track #2)
    publish(AUDIO_TOPIC, "play_study");
};

document.getElementById("partyBtn").onclick = function() {
    console.log("Party Mode Activated");
    // Lights: Rainbow effect
    publish(LIGHTS_TOPIC, '{"effect":"rainbow"}');
    // Video: Turn TV off
    publish(VIDEO_TOPIC, "power_off");
    // Audio: Play party music (track #1)
    publish(AUDIO_TOPIC, "play_party");
};

document.getElementById("offBtn").onclick = function() {
    console.log("All Off");
    publish(LIGHTS_TOPIC, '{"effect":"off"}');
    publish(VIDEO_TOPIC, "power_off");
    publish(AUDIO_TOPIC, "stop");
};