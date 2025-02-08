from flask import Flask, request, jsonify
import requests
import firebase_admin
from firebase_admin import messaging, credentials
from twilio.rest import Client
import smtplib
from email.mime.text import MIMEText

app = Flask(__name__)

# Initialize Firebase Admin
cred = credentials.Certificate("path/to/firebase-adminsdk.json")
firebase_admin.initialize_app(cred)

# Twilio Configuration
TWILIO_ACCOUNT_SID = "your_twilio_account_sid"
TWILIO_AUTH_TOKEN = "your_twilio_auth_token"
TWILIO_PHONE_NUMBER = "your_twilio_phone_number"
twilio_client = Client(TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)

# Email Configuration
SMTP_SERVER = "smtp.your-email-provider.com"
SMTP_PORT = 587
SMTP_USERNAME = "your-email@example.com"
SMTP_PASSWORD = "your-email-password"

def send_push_notification(tokens, title, body):
    message = messaging.MulticastMessage(
        notification=messaging.Notification(title=title, body=body),
        tokens=tokens,
    )
    response = messaging.send_multicast(message)
    return response

def send_sms(phone_number, message):
    twilio_client.messages.create(
        body=message, from_=TWILIO_PHONE_NUMBER, to=phone_number
    )

def send_email(recipient, subject, body):
    msg = MIMEText(body)
    msg["Subject"] = subject
    msg["From"] = SMTP_USERNAME
    msg["To"] = recipient
    
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(SMTP_USERNAME, SMTP_PASSWORD)
        server.sendmail(SMTP_USERNAME, recipient, msg.as_string())

@app.route("/send_alert", methods=["POST"])
def send_alert():
    data = request.json
    user_location = data.get("location")
    user_name = data.get("name")
    phone_number = data.get("phone")
    photo_url = data.get("photo_url")
    
    responders = fetch_nearby_responders(user_location)
    
    title = "🚨 Emergency Alert!"
    body = f"{user_name} is in distress! Location: {user_location}"
    
    for responder in responders:
        if "push_token" in responder:
            send_push_notification([responder["push_token"]], title, body)
        if "phone" in responder:
            send_sms(responder["phone"], body)
        if "email" in responder:
            send_email(responder["email"], title, body)
    
    return jsonify({"status": "Alert sent successfully!"})

def fetch_nearby_responders(location):
    # Simulated database fetch
    return [
        {"name": "Officer A", "phone": "+919876543210", "email": "officer@example.com"},
        {"name": "Government Official B", "push_token": "sample_firebase_token"},
    ]

if __name__ == "__main__":
    app.run(debug=True)
