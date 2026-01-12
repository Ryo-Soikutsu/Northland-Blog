# Challenge 2: Directory Traversal

As mentioned in the previous section, we'll be creating a challenge focused around directory traversal. To complete this challenge, the attacker must access one of the Foundation's secret pages, containing all of the safehouses and facilities that they own.&#x20;

### What is directory traversal?

A directory traversal (or path traversal, as OWASP calls it) attack attempts to access files and directories that are stored outside of the web root folder. These files can contain source code, database schemas or even password files.&#x20;

Directory traversal is performed by manipulating variables to include parent directory references (the dot-dot-slash "../" characters), or absolute file references (e.g. "\~/etc/passwd").&#x20;

Example of a directory traversal attack

{% code overflow="wrap" %}
```
http://some-site.com/get-file?file=rules.html -- A legitimate URL
http://some-site.com/get-file?file=../../../../../etc/passwd -- An exploit, with parent directory references
http://some-site.com/get-file?file=/etc/passwd - Same exploit, but with absolute path
```
{% endcode %}

Depending on how the query is parsed, you may be able to insert files and scripts from another website, to run malicious code



### Creating the vulnerable system (and the vulnerability)

To start, lets create a simple page to indicate to the attacker that they have accessed the correct page

```html
<-- \templates\protected\facilities\earth.html -->
<-- This is just a temporary placeholder page while we develop the challenge, you can edit this however you like -->
{% extends 'base.html' %}

{% block head %}
<link rel="stylesheet" href="{{ url_for('static', filename='/css/facilities.css') }}">
{% endblock %}

{% block title %}Facilities and Safehouses{% endblock %}


{% block content %}
secret page
{% endblock %}


{% block scripts %}
{% endblock %}

```

To make this challenge work, we'll have to add some code to our routes.py script

```python
# core/routes.py
@app.route("/query", methods=["POST"])
def query():
    if request.method == "POST":
        page = request.form["page"]
        if page:
            return render_template(page)
```

We can check that our page works by starting up our Flask server and curling the page

<figure><img src="../.gitbook/assets/image (39).png" alt="" width="375"><figcaption><p>Starting up the Flask server</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (41).png" alt="" width="375"><figcaption><p>Curling the query endpoint</p></figcaption></figure>

On paper, this is the end of the challenge guide. We have an endpoint that accepts user input, and opens the respective page on it.&#x20;

However, there isn't really a way for the attacker to figure out that there is a query endpoint, outside of pure brute forcing of the URL. To fix this, we can rewrite the buttons in our index page to query the new endpoint.&#x20;

```html
<div class="col-sm-4 project-1 project-content">
    <p class="project h6">
                The Sanctuary super-fleet, launched in colaboration with the
                Trans-Supercluster Research and Expeditions Committee (TSREC), spearheads Foundation operations with its
                flagship supercarrier and support dreadnoughts and carriers. <br>
        <button id="proj-1" class="btn btn-secondary m-3 rounded learn-more">Learn More</button>
    </p>
</div>
<-- Repeat for the other buttons in templates\index.html -->
...
<script src="{{ url_for('static', filename="/js/query.js") }}"></script>
```

```python
# core/routes.py
@app.route("/query", methods=["GET"])
def query():
    if request.method == "GET":
        page = request.args.get("page")
        if page:
            try:
                return render_template(page)
            except exceptions.TemplateNotFound as e:
                referrer = request.referrer
                return redirect(referrer)
    return redirect("/")
```

We need to add a JS script too to enable the event listeners on the buttons

```javascript
// static/js/query.js
// Select all buttons with the class 'get-button'
const buttons = document.querySelectorAll('.learn-more');

// Add event listeners to each button
buttons.forEach(button => {
    button.addEventListener('click', function() {
        const page = button.getAttribute('id');
        console.log(page)

        const url = `/query?page=${encodeURIComponent(page)}`;

        // Redirect the user to the new URL
        window.location.href = url;
    });
});

// don't forget to import it
// use <script src="{{ url_for('static', filename="/js/query.js") }}"></script> to import
```

Performing our test in Burp Suite, we can see the URL that the button will query when you click on it

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

We can modify our page parameter to point to our protected page

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption><p>Remember to URL encode!</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption><p>Our response came back!</p></figcaption></figure>

And there we have it! A basic directory traversal vulnerability that the attacker can exploit to gain access to the Foundation's top secret facilities. However, since the page cannot be discovered through normal manipulation of the URL endpoint, we'll have to implement some method of letting attackers know of this secret page.

One way we can do this is through the robots.txt page, which most websites have. To keep it brief, a robots.txt file tells web crawlers which pages and subdirectories they are allowed or not allowed to access. Typically, administrative pages will be added to this robots.txt and marked as "Disallow"&#x20;

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption><p>An example of a robots.txt file</p></figcaption></figure>

To implement this, we can use static files and Flask's send\_from\_directory() function

```html
<-- static\robots.txt -->
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /protected/
Disallow: /protected/facilities
```

```python
# core/routes.py
@app.route("/robots.txt")
def static_from_root():
    return send_from_directory(app.static_folder, request.path[1:])
```

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption><p>curl the new endpoint to test that it is working</p></figcaption></figure>

And we're done!

### Protecting actual files

Unfortunately, we cannot be certain that the attacker will only attempt to access the "supposed" target. They may decide that they want to directly access our set of flags, perhaps stored in the root directory, or even our login credentials to access the server directly. To protect against this, we can do several things

The simplest method would be to sanitize the user input, to strip any parent directory references. To do so, we can add this code to our route

```python
# core/routes.py
@app.route("/query", methods=["GET"])
def query():
    if request.method == "GET":
        page = request.args.get("page")
        if page:
            if ".." in page or "%2F" in page or page.startswith("/"):
                return redirect("/")
            try:
                return render_template(page)
            except exceptions.TemplateNotFound as e:
                referrer = request.referrer
                return redirect(referrer)
    return redirect("/")
```

This is a relatively simple sanitization method, that prevents the attacker from attempting directory traversal. There are possible workarounds to this, such as double URL encoding ("/" is encoded to "%2F", which is then encoded to "%252F"), so this is by no means bullet-proof.&#x20;

Alternatively, you can set up a web root directory, which dictates which files can be accessed by the public, and keeping your sensitive files outside of the directory. However, we will not be going in depth on this. To learn more, you can refer to the following [resource](https://www.seobility.net/en/wiki/Root_Directory)

### Conclusion

This concludes the directory traversal challenge creation guide. In the next section, we'll be attempting to create a server-side template injection vulnerability, and attempt to get the environmental variables stored on the host server.

The code for this section can be found [here](https://github.com/IronForce-Auscent/Sanctuary-Repository/tree/4b8140154d4a9f4bb451aa10f6ee49b9859ed403)
