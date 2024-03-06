<template>
  <div>
    <!-- Button to start recording -->
    <button @click="toggleRecording">{{ recording ? 'Arrêter l\'enregistrement' : 'Commencer l\'enregistrement' }}</button>

    <!-- Display the recorded audio when available -->
    <audio v-if="audioURL" controls>
      <source :src="audioURL" type="audio/wav">
      Votre navigateur ne supporte pas l'élément audio.
    </audio>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Variables
const recording = ref(false); // Variable to track recording state
let mediaRecorder = null; // Variable to store MediaRecorder instance
let chunks = []; // Array to store recorded audio chunks
const audioURL = ref(''); // URL of the recorded audio file

// Methods
const toggleRecording = () => {
  if (!recording.value) {
    startRecording();
  } else {
    stopRecording();
  }
};

const startRecording = () => {
  // Check if the browser supports the getUserMedia API
  if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
    // Access the user's microphone
    navigator.mediaDevices.getUserMedia({ audio: true })
      .then(stream => {
        // Create a MediaRecorder instance
        mediaRecorder = new MediaRecorder(stream);
        // Reset the chunks array
        chunks = [];
        // Event listener for when data is available
        mediaRecorder.ondataavailable = event => {
          // Push the recorded chunk to the array
          chunks.push(event.data);
        };
        // Event listener for when recording stops
        mediaRecorder.onstop = () => {
          // Concatenate the audio chunks into a single Blob
          const audioBlob = new Blob(chunks, { type: 'audio/wav' });
          // Convert Blob to base64 string
          const reader = new FileReader();
          reader.onload = () => {
            const base64Data = reader.result;
            // Set the audio URL to the base64 data
            audioURL.value = base64Data;
            // Send the base64 audio data to an API for transcription
            transcribeAudio(base64Data);
          };
          reader.readAsDataURL(audioBlob);
        };
        // Start recording
        mediaRecorder.start();
        // Notify the user that recording has started
        console.log('Enregistrement en cours...');
        // Update recording state
        recording.value = true;
      })
      .catch(error => {
        console.error('Error accessing the microphone:', error);
      });
  } else {
    console.error('getUserMedia not supported on your browser');
  }
};

const stopRecording = () => {
  // Stop recording
  mediaRecorder.stop();
  // Notify the user that recording has stopped
  console.log('Enregistrement arrêté.');
  // Update recording state
  recording.value = false;
};

const transcribeAudio = (base64Data) => {
  // Call your transcription API with the base64 audio data
  // Example:
  // fetch('https://your-transcription-api.com', {
  //   method: 'POST',
  //   body: JSON.stringify({ audio: base64Data }),
  //   headers: {
  //     'Content-Type': 'application/json'
  //   }
  // })
  // .then(response => response.json())
  // .then(data => console.log('Transcription:', data))
  // .catch(error => console.error('Transcription error:', error));
};
</script>