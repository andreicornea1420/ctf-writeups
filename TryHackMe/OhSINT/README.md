# OhSINT

**Platform**: TryHackMe
**Category**: OSINT
**Difficulty**: Easy

**Challenge Description**

<img width="1264" height="139" alt="image" src="https://github.com/user-attachments/assets/671c3162-daa5-4ada-9144-2b50037d9b4c" />
<br>
<img width="1267" height="776" alt="image" src="https://github.com/user-attachments/assets/1b1bf1c4-cbca-4dfd-a0cc-eb89e1d373da" />

## Solving

<img width="1920" height="1080" alt="WindowsXP_1551719014755" src="https://github.com/user-attachments/assets/3f86fd71-ffc1-46d8-b09a-be4f605307e9" />

This is the image I received initially.

### Q1: What is this user's avatar of?

By checking the metadata with the `exiftool image_name` command, I discovered a username: OWoodflint. <br><br>
<img width="715" height="524" alt="image" src="https://github.com/user-attachments/assets/0e576406-f885-42f8-a2dc-34a176bbe9e1" />

If I search online, I find an account of his on X where he has a picture with a <strong>cat</strong>.

<img width="757" height="273" alt="image" src="https://github.com/user-attachments/assets/cb50df06-99d3-4ddc-9c93-e86f88908c02" />
<br>
A: cat

### Q2 & Q3: What city is this person in? & What is the SSID of the WAP he connected to?

There is a post on their X account featuring the BSSID: the MAC address specific to the router or access point to which the device is connected.

Now I can go to a site like <a href=https://wigle.net>wigle.net</a> and look up the location using this BSSID (I need to be registered first).

A purple dot appears on the map in London here, and if I tap on it, I can also find the SSID: UnileverWiFi.

A3: London
<br>
A4: UnileverWiFi

### Q4 & Q5: What is his personal email address? & What site did you find his email address on?

Besides X, a repository from his GitHub page also shows up on Google.

<img width="949" height="603" alt="image" src="https://github.com/user-attachments/assets/17974f2f-00d4-47ea-945d-304550c7ef3e" />

A4: OWoodflint@gmail.com
<br>
A5: GitHub

### Q6: Where has he gone on holiday?

There is a link on the GitHub repository leading to a WordPress page.

<img width="1130" height="673" alt="image" src="https://github.com/user-attachments/assets/596b88b2-887a-4b17-895a-939f96669ca2" />

A6: New York

### Q7: What is the person's password?

I notice a space between the end of the text and the bottom, so I decided to hover over it and found a string that looks like a password.

<img width="852" height="339" alt="image" src="https://github.com/user-attachments/assets/9e80c901-01fb-45c8-a58e-654143034e7f" />

A7: pennYDr0pper.!

## Lesson Learned

<ul>
  <li>Always check socials and blogs very carefully</li>
  <li>Extract any information from OSINT, no matter how insignificant it may seem.</li>
</ul>



