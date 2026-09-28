# Human Detection in Search and Rescue

A web-based system that helps rescue operations detect people in live camera feeds and uploaded videos, and lets an admin manage rescue teams and assign work to them. A companion Android app gives rescue team members access to alerts and assignments on their phones.

This was a team project. My part was the **Django admin module** and the **Android app** (see the companion repository below).

## Features

**Admin (web)**
- Secure login and password change, plus an OTP-based forgot-password flow sent by email
- Register, edit and delete rescue teams
- Allocate work to a rescue team with details, place and map coordinates
- Track work status (pending, completed, paid) and record offline or online payments
- View feedback submitted by rescue teams
- Run human detection on a live camera feed or on an uploaded video

**Detection**
- Uses a pre-trained YOLO model (Darknet config, COCO class labels) through OpenCV's DNN module
- Keeps only the `person` class, draws bounding boxes with confidence scores, and shows an on-screen alert
- Saves flagged frames and logs each detection (timestamp and image path) to MySQL
- Plays an alert sound when a person is detected

**Rescue team (Android app)**
- Log in, view detected-human alerts with images, view assigned work, update status, send feedback, change password

## Tech stack

| Area | Tools |
|---|---|
| Backend | Python, Django |
| Database | MySQL |
| Detection | OpenCV (DNN), NumPy, pygame (alert sound), pre-trained YOLO |
| Frontend | HTML, CSS, JavaScript, Bootstrap |
| Mobile | Java, Android Studio, Volley, Picasso |

## How it works

1. The admin adds rescue teams and allocates work from the web dashboard.
2. Detection runs on a camera feed or uploaded video. When a person is found, the frame is saved and a record is written to the `detection` table.
3. The Android app talks to the Django backend over HTTP, so rescue teams see the latest detections and their assigned work.

## Project structure

```
Detect_Hu_In_SAR/            Django project settings and URLs
Detect_Hu_In_SAR_app/        Views, models, static files
  static/yolo_files/         yolo.cfg and coco.names (weights not included)
templates/                   HTML templates (admin pages, login, OTP, upload)
DBConnection.py              MySQL helper used by the detection scripts
real_detection.py            Live camera detection
newcam.py                    Detection on uploaded video
manage.py
```

## Setup

Developed with Python 3.6 and Django 2.x on Windows.

1. **Clone and create a virtual environment**
   ```bash
   git clone https://github.com/Abhi-ram-S-IN/<repo-name>.git
   cd <repo-name>
   python -m venv venv
   venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Create the database** in MySQL:
   ```sql
   CREATE DATABASE human_dt_sar;
   ```

3. **Set environment variables** (do not hard-code these):
   ```
   DJANGO_SECRET_KEY=<any long random string>
   SAR_DB_PASSWORD=<your MySQL password>
   SAR_EMAIL=<gmail address used to send OTP emails>
   SAR_EMAIL_PASSWORD=<Gmail app password>
   ```

4. **Add the YOLO weights.** The weights file is too large for GitHub. Download the weights that match `yolo.cfg` and save them as:
   ```
   Detect_Hu_In_SAR_app/static/yolo_files/yolo.weights
   ```

5. **Update file paths.** `real_detection.py`, `newcam.py` and `views.py` currently contain absolute Windows paths to the model files. Change them to point to your local project folder.

6. **Run migrations and start the server**
   ```bash
   python manage.py migrate
   python manage.py runserver
   ```

7. **Create the first admin.** Insert a row into the `Detect_Hu_In_SAR_app_login` table with `usertype = 'admin'` and a username and password of your choice, then log in at `http://127.0.0.1:8000/`.

## Screenshots

_Add screenshots of the admin dashboard, work allocation page and detection output here._

## Known limitations and future improvements

- Detection uses an off-the-shelf COCO model. It has not been fine-tuned on aerial or search-and-rescue imagery, so accuracy on drone footage will be limited.
- Passwords are stored as plain text in the `login` table; they should be hashed (for example with Django's `make_password`).
- Model and video paths are hard-coded and should move to settings.
- Detection runs in a blocking loop for up to one minute per run; a task queue or streaming approach would scale better.

## Companion Android app

The rescue team app lives in a separate repository: `<link-to-android-repo>`

## Author

**Abhiram S**: [LinkedIn](https://linkedin.com/in/abhiram-s-93379628b) | [GitHub](https://github.com/Abhi-ram-S-IN)
