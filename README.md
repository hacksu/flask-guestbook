# flask-guestbook

There is an excellent [HacKSU lesson](https://github.com/hacksu/express-guestbook) that I basically re-implemented in Flask for this workshop.

### What is this?

I wanted to give a simple example in Flask of how you would pass data to the backend with a form...... so let's do that.

### HTML

Let's start with the HTML. We need to be able to take some input from the user and send that to the backend. The best way to do this in HTML5 is with forms.

First, we create a `templates/` folder in the root directory of our project. This is where all of our HTML will go (this is where Flask will recognize it).

I made `book.html`, which looks like this:

```HTML
<!-- Explain this code -->
<!doctype html>
<title>Guestbook</title>

<h1>Welcome to the epic Guestbook!</h1>

<form method="post">
  <input type="text" placeholder="write a message..." name="entryText" />
  <button type="submit">Add Guestbook Entry</button>
</form>
```


That's wonderful. That's all the HTML we're going to need for now.

### Flask

Flask requires a little bit of boilerplate and by a little bit, I mean relatively none compared to other frameworks.

```Python
# Explain this code
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
    return render_template("book.html", entries=entries)
```

Wow. Wonderful. 6 lines of beautiful Python and it just.. works.
If we run the `flask --app app run` command, we get a "nice" result.

### Let's go crazy

Now, we need to make it interactive. Currently if we type some text in the input box and click submit, it does absolutely nothing.

We need to update our Flask code with a new couple more lines:
```Python
# Explain changes
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)

@app.route("/")
def home():
    return render_template("book.html", entries=entries)

entries = []

@app.route("/add", methods=["POST"])
def add_entry():
    entries.append(request.form["entryText"])
    return redirect(url_for('home'))
```

We also have to update our HTML:
```HTML
<!-- Explain this new HTML -->
<!doctype html>
<title>Guestbook</title>

<h1>Welcome to the epic Guestbook!</h1>

<p>Previous entries:</p>
<div>
  {% for entry in entries %}
    <p>{{ entry }}</p>
  {% endfor %}
</div>

<form method="post" action="/add">
  <input type="text" placeholder="write a message..." name="entryText" />
  <button type="submit">Add Guestbook Entry</button>
</form>
```

Now if we type in the box and click the add button, we should see the entry appear on the page.

### Where to go from here?

Well currently, if one user adds something to the page and the other user's don't refresh,
they won't see the changes.

Why don't we fix this problem? Let's change up how our endpoint works:
```Python
from flask import Flask, render_template, request, redirect, url_for
from markupsafe import Markup

app = Flask(__name__)
entries = []

@app.route("/")
def home():
    return render_template("book.html", entries=entries)

@app.route("/add", methods=["POST"])
def add_entry():
    entries.append(request.form["entryText"])
    return redirect(url_for('home'))

# Explain what we are doing here
@app.route("/entries")
def get_all_entries():
    return [Markup.escape(x) for x in entries]
```

And then let's add some JavaScript to our template:
```HTML

<!doctype html>
<title>Guestbook</title>

<h1>Welcome to the epic Guestbook!</h1>

<p>Previous entries:</p>
<div id="entries">
</div>

<form method="post" action="/add">
  <input type="text" placeholder="write a message..." name="entryText" />
  <button type="submit">Add Guestbook Entry</button>
</form>

<!-- REALLY EXPLAIN THIS -->
<script>
const entries = document.getElementById("entries");

function update() {

  for (const child of entries.children) {
    entries.innerHTML = '';
  }

  (async () => {
    const response = await fetch("/entries").then(x => x.json()).catch(_ => null);
    if (response) {
      for (const entry of response) {
        const element = document.createElement("p");
        element.innerHTML = entry;
        entries.appendChild(element);
      }
    }
  })();

}

setInterval(update, 5000);
update();

</script>
```

And there we go, if we spin up multiple sessions, we can see the page is correctly updating on both pages.

### That's really ugly

Yeah. I know it is. This is why we use frameworks like React or Vue to manage our page state instead of doing it by ourselves with raw JavaScript. While it is possible, it is unpleasant to do along with the added fact that my version above adds some page flicker.