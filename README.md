AI-YouTube-AutoUploader/
│
├── audio/
│   ├── voiceover.mp3
│   └── bg_music.mp3
│
├── images/
│   ├── scene_1.png
│   ├── scene_2.png
│   └── scene_3.png
│
├── thumbnails/
│   └── thumbnail.png
│
├── videos/
│   └── final_video.mp4
│
├── scripts/
│   ├── script_generator.py
│   ├── tts_generator.py
│   ├── image_generator.py
│   ├── video_creator.py
│   └── thumbnail_generator.py
│
├── uploader/
│   └── youtube_uploader.py
│
├── utils/
│   ├── config.py
│   ├── logger.py
│   ├── prompts.py
│   └── helpers.py
│
├── main.py
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
    YOUTUBE_CLIENT_SECRET = os.getenv("YOUTUBE_CLIENT_SECRET")
    VIDEO_TYPE = os.getenv("VIDEO_TYPE", "long")  # short | long
    VIDEO_DURATION = int(os.getenv("VIDEO_DURATION", 480))  # seconds
    OUTPUT_DIR = os.getenv("OUTPUT_DIR", "videos")import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger("AI-YT-AUTO")VIDEO_SCRIPT_PROMPT = """
Write a YouTube video script in Hindi.
Topic: {topic}
Duration: {duration} seconds
Style: engaging, storytelling, simple language
Include hook, main content, and ending CTA.
"""

IMAGE_PROMPT = """
3D cinematic illustration, consistent character design,
dramatic lighting, ultra-detailed, YouTube style
Scene: {scene}
"""from openai import OpenAI
from utils.config import Config
from utils.prompts import VIDEO_SCRIPT_PROMPT

client = OpenAI(api_key=Config.OPENAI_API_KEY)

def generate_script(topic):
    prompt = VIDEO_SCRIPT_PROMPT.format(
        topic=topic,
        duration=Config.VIDEO_DURATION
    )

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}]
    )

    return response.choices[0].message.contentimport subprocess

def generate_voiceover(script_text):
    with open("audio/script.txt", "w", encoding="utf-8") as f:
        f.write(script_text)

    subprocess.run([
        "ffmpeg", "-y",
        "-f", "lavfi",
        "-i", "flite=textfile=audio/script.txt:voice=kal",
        "audio/voiceover.mp3"
    ])from openai import OpenAI
from utils.prompts import IMAGE_PROMPT
from utils.config import Config

client = OpenAI(api_key=Config.OPENAI_API_KEY)

def generate_images(scenes):
    for i, scene in enumerate(scenes, 1):
        prompt = IMAGE_PROMPT.format(scene=scene)

        img = client.images.generate(
            model="gpt-image-1",
            prompt=prompt,
            size="1024x1024"
        )

        with open(f"images/scene_{i}.png", "wb") as f:
            f.write(img.data[0].b64_json.decode("base64"))import subprocess

def create_video():
    subprocess.run([
        "ffmpeg",
        "-y",
        "-r", "1/5",
        "-i", "images/scene_%d.png",
        "-i", "audio/voiceover.mp3",
        "-c:v", "libx264",
        "-vf", "fps=30",
        "-pix_fmt", "yuv420p",
        "videos/final_video.mp4"
    ])from PIL import Image, ImageDraw, ImageFont

def generate_thumbnail(text):
    img = Image.new("RGB", (1280, 720), color="black")
    draw = ImageDraw.Draw(img)

    draw.text((50, 300), text, fill="white")
    img.save("thumbnails/thumbnail.png")from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

def upload_video(title, description):
    youtube = build("youtube", "v3", developerKey="YOUR_API_KEY")

    request = youtube.videos().insert(
        part="snippet,status",
        body={
            "snippet": {
                "title": title,
                "description": description,
                "categoryId": "22"
            },
            "status": {
                "privacyStatus": "public"
            }
        },
        media_body=MediaFileUpload("videos/final_video.mp4")
    )

    request.execute()from scripts.script_generator import generate_script
from scripts.tts_generator import generate_voiceover
from scripts.video_creator import create_video
from scripts.thumbnail_generator import generate_thumbnail
from uploader.youtube_uploader import upload_video
from utils.logger import logger

TOPIC = "राजा और किसान की कहानी"

def run():
    logger.info("Generating script...")
    script = generate_script(TOPIC)

    logger.info("Generating voice-over...")
    generate_voiceover(script)

    logger.info("Creating video...")
    create_video()

    logger.info("Generating thumbnail...")
    generate_thumbnail(TOPIC)

    logger.info("Uploading to YouTube...")
    upload_video(TOPIC, script)

if __name__ == "__main__":
    run()openai
python-dotenv
google-api-python-client
Pillow
ffmpeg-pythonOPENAI_API_KEY=sk-xxxxxxxx
YOUTUBE_CLIENT_SECRET=client_secret.json
VIDEO_TYPE=long
VIDEO_DURATION=480# AI-YouTube-AutoUploader 🚀

An end-to-end AI-powered system that automatically creates and uploads YouTube videos daily — zero manual effort.

## 🔥 Features
- AI script generation
- Text-to-speech voice-over
- AI image generation
- FFmpeg video creation
- Auto thumbnail creation
- Daily auto-upload to YouTube
- Supports Shorts & Long videos
- No copyright music

## 🧠 Workflow
AI → Script → Voice → Images → Video → Thumbnail → Upload → Done

## ⚙️ Installation
```bash
git clone https://github.com/yourname/AI-YouTube-AutoUploader
cd AI-YouTube-AutoUploader
pip install -r requirements.txt
