<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Code Video Call with YouTube</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; }
        #qrcode, #videoCall, #youtubePlayer { margin: 20px 0; }
        video { width: 300px; height: 225px; }
    </style>
</head>
<body>
    <h1>QR Code Video Call with YouTube</h1>
    <div id="qrcode"></div>
    <div id="videoCall">
        <video id="localVideo" autoplay muted></video>
        <video id="remoteVideo" autoplay></video>
    </div>
    <div id="youtubePlayer"></div>

    <script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
    <script>
        let peerConnection;
        let localStream;
        const configuration = {'iceServers': [{'urls': 'stun:stun.l.google.com:19302'}]};

        async function init() {
            try {
                localStream = await navigator.mediaDevices.getUserMedia({video: true, audio: true});
                document.getElementById('localVideo').srcObject = localStream;

                peerConnection = new RTCPeerConnection(configuration);
                localStream.getTracks().forEach(track => peerConnection.addTrack(track, localStream));

                peerConnection.ontrack = event => {
                    document.getElementById('remoteVideo').srcObject = event.streams[0];
                };

                const offer = await peerConnection.createOffer();
                await peerConnection.setLocalDescription(offer);

                const roomId = Math.random().toString(36).substring(7);
                const qr = new QRCode(document.getElementById("qrcode"), {
                    text: `${window.location.href}?room=${roomId}`,
                    width: 128,
                    height: 128
                });

                // Here you would typically send the offer to a signaling server
                console.log('Offer:', offer);
                console.log('Room ID:', roomId);
            } catch (error) {
                console.error('Error:', error);
            }
        }

        // Load YouTube API
        const tag = document.createElement('script');
        tag.src = "https://www.youtube.com/iframe_api";
        const firstScriptTag = document.getElementsByTagName('script')[0];
        firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

        let player;
        function onYouTubeIframeAPIReady() {
            player = new YT.Player('youtubePlayer', {
                height: '360',
                width: '640',
                videoId: 'dQw4w9WgXcQ',
                events: {
                    'onReady': onPlayerReady
                }
            });
        }

        function onPlayerReady(event) {
            // Player is ready
        }

        init();
    </script>
</body>
</html>
    <div id="videoSelector">
        <input type="text" id="videoIdInput" placeholder="Enter YouTube Video ID">
        <button onclick="loadVideo()">Load Video</button>
    </div>

    <script>
        function loadVideo() {
            const videoId = document.getElementById('videoIdInput').value.trim();
            if (videoId) {
                player.loadVideoById(videoId);
            } else {
                alert('Please enter a valid YouTube Video ID');
            }
        }

        // Modify the onYouTubeIframeAPIReady function
        function onYouTubeIframeAPIReady() {
            player = new YT.Player('youtubePlayer', {
                height: '360',
                width: '640',
                videoId: 'dQw4w9WgXcQ', // Default video
                playerVars: {
                    'autoplay': 0,
                    'controls': 1,
                    'rel': 0,
                    'fs': 1
                },
                events: {
                    'onReady': onPlayerReady
                }
            });
        }

        function onPlayerReady(event) {
            // Player is ready
            console.log('YouTube player is ready');
        }

        // Add styles for the video selector
        const style = document.createElement('style');
        style.textContent = `
            #videoSelector {
                margin-top: 20px;
                margin-bottom: 20px;
            }
            #videoIdInput {
                padding: 5px;
                width: 200px;
            }
            #videoSelector button {
                padding: 5px 10px;
                background-color: #4CAF50;
                color: white;
                border: none;
                cursor: pointer;
            }
            #videoSelector button:hover {
                background-color: #45a049;
            }
        `;
        document.head.appendChild(style);
    </script>
    <div id="videoSelector">
        <input type="text" id="videoUrlInput" placeholder="Paste YouTube video URL here">
        <button onclick="loadVideo()">Load Video</button>
    </div>

    <script>
        function loadVideo() {
            const videoUrl = document.getElementById('videoUrlInput').value;
            const videoId = extractVideoId(videoUrl);
            
            if (videoId) {
                player.loadVideoById(videoId);
            } else {
                alert('Invalid YouTube URL. Please try again.');
            }
        }

        function extractVideoId(url) {
            const regExp = /^.*(youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|\&v=)([^#\&\?]*).*/;
            const match = url.match(regExp);
            return (match && match[2].length === 11) ? match[2] : null;
        }

        // Update styles for the new input
        const additionalStyle = document.createElement('style');
        additionalStyle.textContent = `
            #videoUrlInput {
                padding: 5px;
                width: 300px;
                margin-right: 10px;
            }
        `;
        document.head.appendChild(additionalStyle);
    </script>
    <div id="roomControls">
        <input type="text" id="roomIdInput" placeholder="Enter room ID">
        <button onclick="joinRoom()">Join Room</button>
        <button onclick="createRoom()">Create New Room</button>
    </div>

    <script>
        let roomId = null;
        let socket = null;

        function createRoom() {
            roomId = Math.random().toString(36).substring(7);
            initializeSocket();
            alert(`Room created! Share this ID with others: ${roomId}`);
        }

        function joinRoom() {
            roomId = document.getElementById('roomIdInput').value.trim();
            if (roomId) {
                initializeSocket();
            } else {
                alert('Please enter a valid room ID');
            }
        }

        function initializeSocket() {
            // In a real implementation, replace 'your-websocket-server-url' with your actual WebSocket server URL
            socket = new WebSocket('your-websocket-server-url');

            socket.onopen = function(event) {
                console.log('Connected to WebSocket server');
                socket.send(JSON.stringify({type: 'join', roomId: roomId}));
            };

            socket.onmessage = function(event) {
                const data = JSON.parse(event.data);
                if (data.type === 'videoControl') {
                    handleVideoControl(data);
                }
            };

            socket.onerror = function(error) {
                console.error('WebSocket error:', error);
            };

            socket.onclose = function(event) {
                console.log('Disconnected from WebSocket server');
            };
        }

        function handleVideoControl(data) {
            switch(data.action) {
                case 'play':
                    player.playVideo();
                    break;
                case 'pause':
                    player.pauseVideo();
                    break;
                case 'seek':
                    player.seekTo(data.time);
                    break;
            }
        }

        // Modify existing video control functions to broadcast to all users
        function onPlayerStateChange(event) {
            if (socket && socket.readyState === WebSocket.OPEN) {
                if (event.data == YT.PlayerState.PLAYING) {
                    socket.send(JSON.stringify({type: 'videoControl', action: 'play', roomId: roomId}));
                } else if (event.data == YT.PlayerState.PAUSED) {
                    socket.send(JSON.stringify({type: 'videoControl', action: 'pause', roomId: roomId}));
                }
            }
        }

        function seekTo(time) {
            player.seekTo(time);
            if (socket && socket.readyState === WebSocket.OPEN) {
                socket.send(JSON.stringify({type: 'videoControl', action: 'seek', time: time, roomId: roomId}));
            }
        }

        // Add styles for room controls
        const roomControlsStyle = document.createElement('style');
        roomControlsStyle.textContent = `
            #roomControls {
                margin-top: 10px;
            }
            #roomIdInput {
                padding: 5px;
                width: 200px;
                margin-right: 10px;
            }
            #roomControls button {
                padding: 5px 10px;
                background-color: #4CAF50;
                color: white;
                border: none;
                cursor: pointer;
                margin-right: 5px;
            }
            #roomControls button:hover {
                background-color: #45a049;
            }
        `;
        document.head.appendChild(roomControlsStyle);
    </script>
