<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voice Cloning Website</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}">
</head>
<body>
    <header>
        <h1>Welcome to Voice Cloning</h1>
        <p>Apni awaaz ko clone karein aur naye anubhav paayein!</p>
        <button id="startBtn">Shuru Karein</button>
    </header>

    <section id="uploadSection">
        <h2>Apni Audio File Upload Karein</h2>
        <form id="uploadForm">
            <input type="file" id="audioFile" accept="audio/*" required>
            <button type="submit">Upload</button>
        </form>
        <div id="progressBar">
            <div id="progress"></div>
        </div>
    </section>

    <section id="audioPlayerSection">
        <h2>Cloned Audio</h2>
        <audio id="clonedAudio" controls>
            <source src="" type="audio/mpeg">
            Your browser does not support the audio element.
        </audio>
    </section>

    <section id="settingsSection">
        <h2>Settings</h2>
        <form id="settingsForm">
            <label for="audioFormat">Audio Format:</label>
            <select id="audioFormat">
                <option value="mp3">MP3</option>
                <option value="wav">WAV</option>
            </select>

            <label for="audioQuality">Audio Quality:</label>
            <select id="audioQuality">
                <option value="high">High</option>
                <option value="medium">Medium</option>
                <option value="low">Low</option>
            </select>

            <button type="submit">Save Settings</button>
        </form>
    </section>

    <footer>
        <p>&copy; 2025 Voice Cloning Website. Sabhi adhikar surakshit hain.</p>
        <p>Contact: <a href="mailto:info@voicecloning.com">info@voicecloning.com</a></p>
        <div class="social-links">
            <a href="https://facebook.com" target="_blank">Facebook</a>
            <a href="https://twitter.com" target="_blank">Twitter</a>
            <a href="https://instagram.com" target="_blank">Instagram</a>
        </div>
    </footer>

    <script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>/* General Styles */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
    color: #333;
    line-height: 1.6;
}

header {
    text-align: center;
    padding: 50px;
    background-color: #007BFF;
    color: white;
}

section {
    margin: 20px;
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

footer {
    text-align: center;
    padding: 20px;
    background-color: #333;
    color: white;
}

.social-links a {
    margin: 0 10px;
    color: white;
    text-decoration: none;
}

.social-links a:hover {
    color: #007BFF;
}

/* Responsive Design */
@media (max-width: 768px) {
    header h1 {
        font-size: 2rem;
    }

    header p {
        font-size: 1rem;
    }

    section {
        margin: 10px;
        padding: 15px;
    }

    footer {
        position: static; /* Unfix footer on small screens */
    }
}// Home Page - Start Button Functionality
document.getElementById('startBtn').addEventListener('click', () => {
    alert('Shuru karein! Aapka swagat hai Voice Cloning website par.');
});

// Audio Upload Section - Handle File Upload
document.getElementById('uploadForm').addEventListener('submit', async (e) => {
    e.preventDefault();

    const fileInput = document.getElementById('audioFile');
    const progressBar = document.getElementById('progress');
    const clonedAudio = document.getElementById('clonedAudio');

    if (fileInput.files.length === 0) {
        alert('Kripya ek audio file upload karein.');
        return;
    }

    const file = fileInput.files[0];
    const formData = new FormData();
    formData.append('file', file);

    try {
        // Simulate file upload progress
        progressBar.style.width = '0%';
        let progress = 0;
        const interval = setInterval(() => {
            progress += 10;
            progressBar.style.width = `${progress}%`;
            if (progress >= 100) clearInterval(interval);
        }, 200);

        // Send file to the server
        const response = await fetch('/clone', {
            method: 'POST',
            body: formData,
        });

        const result = await response.json();
        if (result.success) {
            alert('Audio file successfully uploaded aur cloned!');
            clonedAudio.src = result.audioUrl;
        } else {
            alert('Error: Audio cloning failed. Kripya phir se koshish karein.');
        }
    } catch (error) {
        console.error('Error:', error);
        alert('An error occurred. Kripya phir se koshish karein.');
    }
});

// Settings Section - Handle Settings Form
document.getElementById('settingsForm').addEventListener('submit', (e) => {
    e.preventDefault();
    const audioFormat = document.getElementById('audioFormat').value;
    const audioQuality = document.getElementById('audioQuality').value;

    // Save settings to localStorage
    localStorage.setItem('audioFormat', audioFormat);
    localStorage.setItem('audioQuality', audioQuality);

    alert(`Settings saved: Format - ${audioFormat}, Quality - ${audioQuality}`);
});

// Load saved settings on page load
window.addEventListener('load', () => {
    const savedFormat = localStorage.getItem('audioFormat');
    const savedQuality = localStorage.getItem('audioQuality');

    if (savedFormat) {
        document.getElementById('audioFormat').value = savedFormat;
    }
    if (savedQuality) {
        document.getElementById('audioQuality').value = savedQuality;
    }
});from flask import Flask, request, jsonify, render_template
import os
from utils import clone_voice

app = Flask(__name__)

# Ensure the "uploads" directory exists
if not os.path.exists('uploads'):
    os.makedirs('uploads')

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/clone', methods=['POST'])
def clone_voice_endpoint():
    try:
        if 'file' not in request.files:
            return jsonify({"success": False, "error": "No file uploaded"})
        
        file = request.files['file']
        if file.filename == '':
            return jsonify({"success": False, "error": "No file selected"})
        
        file_path = os.path.join('uploads', file.filename)
        file.save(file_path)
        
        # Add voice cloning logic
        cloned_audio_path = os.path.join('uploads', 'cloned_' + file.filename)
        clone_voice(file_path, cloned_audio_path)  # Call the cloning function
        
        return jsonify({"success": True, "audioUrl": cloned_audio_path})
    except Exception as e:
        return jsonify({"success": False, "error": str(e)})

if __name__ == '__main__':
    app.run(debug=True)from pydub import AudioSegment

def clone_voice(input_path, output_path):
    """
    Clones the voice by modifying the pitch of the input audio file.
    :param input_path: Path to the input audio file.
    :param output_path: Path to save the cloned audio file.
    """
    try:
        audio = AudioSegment.from_file(input_path)
        cloned_audio = audio._spawn(audio.raw_data, overrides={
            "frame_rate": int(audio.frame_rate * 1.1)  # Increase pitch by 10%
        })
        cloned_audio.export(output_path, format="mp3")
    except Exception as e:
        raise Exception(f"Error in cloning voice: {str(e)}")

Know is it correct
