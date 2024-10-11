import openai
import os
import smtplib
import ssl
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import time

# Get API keys and email credentials from Replit's environment variables
openai.api_key = os.getenv('OPENAI_API_KEY')
openai.organization = os.getenv('OPENAI_ORG_ID')
sender_email = os.getenv('EMAIL')  # Your email address
password = os.getenv('EMAIL_PASSWORD')  # Your app-specific password or normal password
receiver_email = "travis.rc.stone.1984@gmail.com"  # Receiver's email

# Define OpenAI request
response = openai.Completion.create(
    engine="gpt-3.5-turbo",
    prompt="Generate a personalized greeting for Travis.",
    max_tokens=50
)

# Extract the response text from OpenAI
openai_response = response.choices[0].text.strip()

# Create email message
message = MIMEMultipart("alternative")
message["Subject"] = "OpenAI Response for Travis"
message["From"] = sender_email
message["To"] = receiver_email

# Create the body of the email (plain text)
text = f"Hello Travis,\n\nHere is the message from OpenAI:\n\n{openai_response}\n\nBest regards,\nYour Flask App"
part1 = MIMEText(text, "plain")
message.attach(part1)

# Define SMTP settings
smtp_server = "smtp.gmail.com"
context = ssl.create_default_context()

# Define a function to try sending email
def try_sending_email(port, use_tls=True):
    try:
        if use_tls:
            print(f"Trying to connect with TLS on port {port}...")
            with smtplib.SMTP(smtp_server, port) as server:
                server.starttls(context=context)  # Secure the connection
                server.login(sender_email, password)
                server.sendmail(sender_email, receiver_email, message.as_string())
                print("Email sent successfully with TLS!")
                return True
        else:
            print(f"Trying to connect with SSL on port {port}...")
            with smtplib.SMTP_SSL(smtp_server, port, context=context) as server:
                server.login(sender_email, password)
                server.sendmail(sender_email, receiver_email, message.as_string())
                print("Email sent successfully with SSL!")
                return True
    except Exception as e:
        print(f"Failed to send email using port {port} ({'TLS' if use_tls else 'SSL'}). Error: {e}")
        return False

# Try the following steps in order:
attempts = [
    {'port': 587, 'use_tls': True},  # First try TLS (port 587)
    {'port': 465, 'use_tls': False},  # If that fails, try SSL (port 465)
    {'port': 587, 'use_tls': True},  # Retry TLS for a double check (port 587)
    {'port': 465, 'use_tls': False}  # Retry SSL if both fail (port 465)
]

# Iterate over the attempts
for attempt in attempts:
    if try_sending_email(attempt['port'], attempt['use_tls']):
        break  # If email is sent, stop the loop
    time.sleep(2)  # Small delay between retries

print("All attempts completed.")

<!---
Thetheorizingstone/Thetheorizingstone is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
