## PDF to Podcast (PDF to Audio Converter)

### Install Dependencies  
```bash
pip install streamlit PyMuPDF gtts

### Pythn code
import streamlit as st
import PyPDF2
import os
import time
import tempfile
from gtts import gTTS
import pyttsx3  # Offline TTS alternative

def extract_text_from_pdf(pdf_file):
    """Extract text from a PDF file."""
    pdf_reader = PyPDF2.PdfReader(pdf_file)
    text = ""
    for page in pdf_reader.pages:
        extracted_text = page.extract_text()
        if extracted_text:
            text += extracted_text + "\n"
    return text if text else None

def text_to_speech_online(text, output_file):
    """Convert text to speech using Google TTS (Online)."""
    try:
        if os.path.exists(output_file):
            os.remove(output_file)  # Delete old file to avoid conflicts
        tts = gTTS(text, lang='en')
        tts.save(output_file)
        time.sleep(1)  # Ensure file is written before accessing it
    except Exception as e:
        st.error(f"Google TTS Error: {e}. Switching to offline TTS.")
        text_to_speech_offline(text, output_file)

def text_to_speech_offline(text, output_file):
    """Convert text to speech using pyttsx3 (Offline)."""
    engine = pyttsx3.init()
    engine.save_to_file(text, output_file)
    engine.runAndWait()

def main():
    st.title("PDF to Podcast ")
    st.write("Upload a research paper PDF and convert it into an audio file.")
    
    uploaded_file = st.file_uploader("Choose a PDF file", type=["pdf"], accept_multiple_files=False)
    
    if uploaded_file is not None:
        text = extract_text_from_pdf(uploaded_file)
        if text:
            st.success("Text extracted successfully!")
            temp_dir = tempfile.gettempdir()
            output_audio = os.path.join(temp_dir, "output_audio.mp3")
            
            # Choose method: Online (gTTS) or Offline (pyttsx3)
            use_online_tts = st.checkbox("Use Google TTS (Online)?", value=True)
            
            if use_online_tts:
                text_to_speech_online(text, output_audio)
            else:
                text_to_speech_offline(text, output_audio)
                
            if os.path.exists(output_audio):
                st.audio(output_audio, format='audio/mp3')
                with open(output_audio, "rb") as file:
                    st.download_button("Download Audio", file, file_name="converted_audio.mp3", mime="audio/mp3")
            else:
                st.error("Audio file was not generated successfully. Please try again.")
        else:
            st.error("Could not extract text from the PDF. Please try another file.")

if __name__ == "__main__":
    main()

### Run it As
streamlit run pdf_to_audio_converter.py
