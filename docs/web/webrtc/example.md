# Example

## Starting a call

```js
function invite(evt) {
  if (myPeerConnection) {
    alert("You can't start a call because you already have one open!");
  } else {
    createPeerConnection();

    navigator.mediaDevices.getUserMedia({
      audio: true, // We want an audio track
      video: true // ...and we want a video track
    })
    .then(function(localStream) {
      document.getElementById("local_video").srcObject = localStream;
      localStream.getTracks().forEach(track => myPeerConnection.addTrack(track, localStream));
    })
    .catch(handleGetUserMediaError);
  }
}


function handleGetUserMediaError(e) {
  switch(e.name) {
    case "NotFoundError":
      alert("Unable to open your call because no camera and/or microphone" +
            "were found.");
      break;
    case "SecurityError":
    case "PermissionDeniedError":
      // Do nothing; this is the same as the user canceling the call.
      break;
    default:
      alert("Error opening your camera and/or microphone: " + e.message);
      break;
  }

  closeVideoCall();
}
```

In the above example, we create peer connection via `createPeerConnection` method. Once the RTCPeerConnection has been created, we request access to the user's camera and microphone by calling `MediaDevices.getUserMedia()` which is exposed to us through the `MediaDevices.getUserMedia` property.

We attach the incoming stream to the local preview `<video>` element by setting the element's srcObject property.

We then iterate over the tracks in the stream, calling `addTrack()` to add each track to the `RTCPeerConnection`. Even though the connection is not fully established yet, you can begin sending data when you feel it's appropriate to do so. Media received before the ICE negotiation is completed may be used to help ICE decide upon the best connectivity approach to take, thus aiding in the negotiation process.

Note that for native apps, such as a phone application, you should not begin sending until the connection has been accepted at both ends, at a minimum, to avoid inadvertently sending video and/or audio data when the user isn't prepared for it.

## Creating the peer connection

```js
function createPeerConnection() {
  myPeerConnection = new RTCPeerConnection({
      iceServers: [     // Information about ICE servers - Use your own!
        {
          urls: "stun:stun.stunprotocol.org"
        }
      ]
  });

  myPeerConnection.onicecandidate = handleICECandidateEvent;
  myPeerConnection.ontrack = handleTrackEvent;
  myPeerConnection.onnegotiationneeded = handleNegotiationNeededEvent;
  myPeerConnection.onremovetrack = handleRemoveTrackEvent;
  myPeerConnection.oniceconnectionstatechange = handleICEConnectionStateChangeEvent;
  myPeerConnection.onicegatheringstatechange = handleICEGatheringStateChangeEvent;
  myPeerConnection.onsignalingstatechange = handleSignalingStateChangeEvent;
}
```

The createPeerConnection() function is used by both the caller and the callee to construct their RTCPeerConnection objects, their respective ends of the WebRTC connection.

When using the RTCPeerConnection() constructor, we will specify an RTCConfiguration-compliant object providing configuration parameters for the connection. This is an array of objects describing STUN and/or TURN servers for the ICE layer to use when attempting to establish a route between the caller and the callee.

After creating the RTCPeerConnection, we set up handlers for the events that matter to us. The first three of these event handlers are required; you have to handle them to do anything involving streamed media with WebRTC.

## Starting negotiation

Once the caller has created its  `RTCPeerConnection`, created a media stream, and added its tracks to the connection, the browser will deliver a `negotiationneeded` event to the `RTCPeerConnection` to indicate that it's ready to begin negotiation with the other peer

```js
function handleNegotiationNeededEvent() {
  myPeerConnection.createOffer().then(function(offer) {
    return myPeerConnection.setLocalDescription(offer);
  })
  .then(function() {
    sendToServer({
      name: myUsername,
      target: targetUsername,
      type: "video-offer",
      sdp: myPeerConnection.localDescription
    });
  })
  .catch(reportError);
}
```

To start the negotiation process, we need to create and send an SDP offer to the peer we want to connect to. This offer includes a list of supported configurations for the connection, including information about the media stream we've added to the connection locally, and any ICE candidates gathered by the ICE layer already. We create this offer by calling `myPeerConnection.createOffer()`.

When `createOffer()` succeeds, we pass the created offer information into `myPeerConnection.setLocalDescription()`, which configures the connection and media configuration state for the caller's end of the connection.

We know the description is valid, and has been set, this is when we send our offer to the other peer by creating a new "video-offer" message containing the local description, then sending it through our signaling server to the callee.

## Handling the invitation

When the offer arrives, the callee's `handleVideoOfferMsg()` function is called with the "video-offer" message that was received

```js
function handleVideoOfferMsg(msg) {
  var localStream = null;

  targetUsername = msg.name;
  createPeerConnection();

  var desc = new RTCSessionDescription(msg.sdp);

  myPeerConnection.setRemoteDescription(desc).then(function () {
    return navigator.mediaDevices.getUserMedia(mediaConstraints);
  })
  .then(function(stream) {
    localStream = stream;
    document.getElementById("local_video").srcObject = localStream;

    localStream.getTracks().forEach(track => myPeerConnection.addTrack(track, localStream));
  })
  .then(function() {
    return myPeerConnection.createAnswer();
  })
  .then(function(answer) {
    return myPeerConnection.setLocalDescription(answer);
  })
  .then(function() {
    var msg = {
      name: myUsername,
      target: targetUsername,
      type: "video-answer",
      sdp: myPeerConnection.localDescription
    };

    sendToServer(msg);
  })
  .catch(handleGetUserMediaError);
}
```

This code is very similar to what we did in the `invite()` function.  It starts by creating and configuring an RTCPeerConnection using our createPeerConnection() function. Then it takes the SDP offer from the received "video-offer" message and uses it to create a new RTCSessionDescription object representing the caller's session description.

That session description is then passed into myPeerConnection.setRemoteDescription(). This establishes the received offer as the description of the remote (caller's) end of the connection.

Once the answer has been created using myPeerConnection.createAnswer(), the description of the local end of the connection is set to the answer's SDP by calling myPeerConnection.setLocalDescription(), then the answer is transmitted through the signaling server to the caller to let them know what the answer is.

## Sending ICE candidates

The ICE negotiation process involves each peer sending candidates to the other, repeatedly, until it runs out of potential ways it can support the `RTCPeerConnection`'s media transport needs.

Once `setLocalDescription()`'s fulfillment handler has run, the ICE agent begins sending `icecandidate` events to the `RTCPeerConnection` one for each potential configuration it discovers.

```js
function handleICECandidateEvent(event) {
  if (!e.candidate) {
    // All candidates have been sent
    return
  }

  sendToServer({
    type: "new-ice-candidate",
    target: targetUsername,
    candidate: event.candidate
  });
}
```

## Receiving ICE candidates

```js
function handleNewICECandidateMsg(msg) {
  var candidate = new RTCIceCandidate(msg.candidate);

  myPeerConnection.addIceCandidate(candidate)
    .catch(reportError);
}
```

This function constructs an `RTCIceCandidate` object by passing the received SDP into its constructor, then delivers the candidate to the ICE layer by passing it into `myPeerConnection.addIceCandidate()`.

## Receiving new streams

When new tracks are added to the `RTCPeerConnection`— either by calling its `addTrack()` method or because of renegotiation of the stream's format—a track event is set to the `RTCPeerConnection` for each track added to the connection. 

```js
function handleTrackEvent(event) {
  document.getElementById("received_video").srcObject = event.streams[0];
  document.getElementById("hangup-button").disabled = false;
}
```

The incoming stream is attached to the "received_video" `<video>` element.

## Handling the removal of tracks

Your code receives a `removetrack` event when the remote peer removes a track from the connection by calling RTCPeerConnection.removeTrack(). 

```js
function handleRemoveTrackEvent(event) {
  var stream = document.getElementById("received_video").srcObject;
  var trackList = stream.getTracks();

  if (trackList.length == 0) {
    closeVideoCall();
  }
}
```

## Ending the call

When the user clicks the "Hang Up" button to end the call, the hangUpCall() function is called:

```js
function hangUpCall() {
  closeVideoCall();
  sendToServer({
    name: myUsername,
    target: targetUsername,
    type: "hang-up"
  });
}
```

```js
function closeVideoCall() {
  var remoteVideo = document.getElementById("received_video");
  var localVideo = document.getElementById("local_video");

  if (myPeerConnection) {
    myPeerConnection.ontrack = null;
    myPeerConnection.onremovetrack = null;
    myPeerConnection.onremovestream = null;
    myPeerConnection.onicecandidate = null;
    myPeerConnection.oniceconnectionstatechange = null;
    myPeerConnection.onsignalingstatechange = null;
    myPeerConnection.onicegatheringstatechange = null;
    myPeerConnection.onnegotiationneeded = null;
    if (remoteVideo.srcObject) {
      remoteVideo.srcObject.getTracks().forEach(track => track.stop());
    }

    if (localVideo.srcObject) {
      localVideo.srcObject.getTracks().forEach(track => track.stop());
    }

    myPeerConnection.close();
    myPeerConnection = null;
  }

  remoteVideo.removeAttribute("src");
  remoteVideo.removeAttribute("srcObject");
  localVideo.removeAttribute("src");
  remoteVideo.removeAttribute("srcObject");

  document.getElementById("hangup-button").disabled = true;
  targetUsername = null;
}
```

## Dealing with state changes

### ICE connection state

`iceconnectionstatechange` events are sent to the `RTCPeerConnection` by the ICE layer when the connection state changes (such as when the call is terminated from the other end).

```js
function handleICEConnectionStateChangeEvent(event) {
  switch(myPeerConnection.iceConnectionState) {
    case "closed":
    case "failed":
      closeVideoCall();
      break;
  }
}
```

### ICE signaling state

```js
function handleSignalingStateChangeEvent(event) {
  switch(myPeerConnection.signalingState) {
    case "closed":
      closeVideoCall();
      break;
  }
};
```

### ICE gathering state

`icegatheringstatechange` events are used to let you know when the ICE candidate gathering process state changes

```js
function handleICEGatheringStateChangeEvent(event) {
  // Our sample just logs information to console here,
  // but you can do whatever you need.
}
```
