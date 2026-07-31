# SnapClass

SnapClass is an AI-powered classroom attendance app. It replaces roll call with face and voice recognition, so teachers can mark attendance in seconds instead of minutes.

**Live app:** https://snapclass-intelligentai-attendance.streamlit.app

## What it does

Teachers create a subject, enroll students by photo, and take attendance with a single class photo or a short audio clip. The app finds every student it recognizes and marks them present.

- **Face attendance**: upload a photo of the class, and SnapClass detects each face, matches it against enrolled students, and marks them present.
- **Voice attendance**: upload a class recording, and SnapClass splits it into speech segments and matches each voice against enrolled students.
- **Auto-enroll by link**: teachers generate a join code or QR code, students scan it, and it enrolls them automatically.
- **Teacher and student roles**: separate screens for creating subjects, managing rosters, and viewing attendance history.

## How the recognition works

**Face pipeline** (`src/pipelines/face_pipeline.py`)
`dlib`'s HOG face detector finds faces in a photo, and its ResNet-based face recognition model turns each one into a 128-dimension embedding. A linear SVM, trained on the fly from enrolled students' stored embeddings, classifies each detected face.

**Voice pipeline** (`src/pipelines/voice_pipeline.py`)
`librosa` splits an audio clip into speech segments by silence detection. `resemblyzer`'s voice encoder turns each segment into an embedding, which gets compared to enrolled students' voice embeddings with cosine similarity. A match only counts above a 0.65 similarity threshold.

## Tech stack

- **Frontend**: Streamlit
- **Face recognition**: dlib, face_recognition_models, scikit-learn (SVM)
- **Voice recognition**: resemblyzer, librosa
- **Database & auth**: Supabase (Postgres), bcrypt for password hashing
- **QR codes**: segno

## Project structure

```
app.py                          # entry point, routes between teacher/student/home screens
src/
├── components/                 # dialogs and widgets (enroll, add photo, voice attendance, etc.)
├── database/                   # Supabase client and queries (config.py, db.py)
├── pipelines/                  # face_pipeline.py, voice_pipeline.py
├── screens/                    # home_screen.py, teacher_screen.py, student_screen.py
└── ui/                         # shared layout
```

## Running it locally

1. Clone the repo and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Set up a Supabase project with `teachers` and `students` tables, and add your credentials to `src/database/config.py` (or as environment variables, depending on your setup).
3. Run the app:
   ```bash
   streamlit run app.py
   ```

## Notes

- Face embeddings and voice embeddings are stored per student in Supabase, so recognition accuracy improves as more enrollment photos and voice samples are added.
- The SVM classifier retrains automatically whenever a new student enrolls.
