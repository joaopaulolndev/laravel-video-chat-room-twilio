<template>
    <div className="p-5">
        <h1 className="text-2xl mb-4">Laravel Video Chat</h1>
        <div className="grid grid-flow-row grid-cols-3 grid-rows-3 gap-4 bg-black">
            <div id="video-chat-window"></div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'video-chat',
    data() {
        return {
            accessToken: ''
        }
    },
    mounted() {
        console.log('Video chat room loading...')

        this.getAccessToken()
    },
    methods: {
        async getAccessToken() {
            // Request a new token
            const response = await axios.get('/api/access_token');

            this.accessToken = response.data;
            this.connectToRoom();
        },
        connectToRoom() {

            const {connect, createLocalVideoTrack} = require('twilio-video');

            connect(this.accessToken, {name: 'cool room'}).then(async room => {

                console.log(`Successfully joined a Room: ${room}`);

                const videoChatWindow = document.getElementById('video-chat-window');

                createLocalVideoTrack().then(track => {
                     videoChatWindow.appendChild(track.attach());
                });

                room.on('participantConnected', participant => {
                    console.log(`Participant "${participant.identity}" connected`);

                    participant.tracks.forEach(publication => {
                        if (publication.isSubscribed) {
                            const track = publication.track;
                            videoChatWindow.appendChild(track.attach());
                        }
                    });

                    participant.on('trackSubscribed', track => {
                        videoChatWindow.appendChild(track.attach());
                    });
                });
            }, error => {
                console.error(`Unable to connect to Room: ${error.message}`);
            });
        }
    }
}
</script>
