# amazon-keyword-link-software-new-
3 keywords and open Google + Amazon India product cable
from flask import Flask, request, redirect, url_for, render_template_string
import itertools
import urllib.parse

app = Flask(__name__)

# Hard-coded three keywords (no user input for keywords)
KEYWORDS = ["red shoes for men", "sports running shoes", "comfortable sneakers"]

# Round-robin iterator over keywords
keyword_cycle = itertools.cycle(KEYWORDS)

# Simple in-memory mapping: token -> amazon product URL
# In real use, you would store this in a database
PRODUCT_LINKS = {}

# Simple page to create a link for a specific Amazon product page
# You only input the specific product page URL here
CREATE_PAGE = """
<!doctype html>
<html>
  <head><title>Create Redirect Link</title></head>
  <body>
    <h1>Create Redirect Link</h1>
    <form method="post">
      <label>Amazon India product URL:</label><br>
      <input type="text" name="amazon_url" size="80" required><br><br>
      <button type="submit">Create Link</button>
    </form>
    {% if link %}
      <p>Your customer link:</p>
      <p><a href="{{ link }}" target="_blank">{{ link }}</a></p>
    {% endif %}
  </body>
</html>
"""

# HTML used after choosing keyword; shows Google first, then redirects to Amazon product
REDIRECT_PAGE_TEMPLATE = """
<!doctype html>
<html>
  <head>
    <title>Redirecting...</title>
    <meta charset="utf-8">
    <script>
      // After delay, redirect to Amazon product page
      setTimeout(function() {
        window.location.href = "{{ amazon_url }}";
      }, 3000);  // 3 seconds on Google before jumping to Amazon
    </script>
  </head>
  <body>
    <p>Opening Google search for: <strong>{{ keyword }}</strong>...</p>
    <iframe src="{{ google_url }}" style="width:100%; height:80vh; border:none;"></iframe>
  </body>
</html>
"""

@app.route("/create", methods=["GET", "POST"])
def create():
    link = None
    if request.method == "POST":
        amazon_url = request.form.get("amazon_url", "").strip()
        if amazon_url:
            # Create a simple token from the product URL
            token = urllib.parse.quote_plus(amazon_url)
            PRODUCT_LINKS[token] = amazon_url
            link = request.host_url.rstrip("/") + url_for("entry", token=token)
    return render_template_string(CREATE_PAGE, link=link)

@app.route("/go/<token>")
def entry(token):
    # Find the Amazon product URL from token
    amazon_url = PRODUCT_LINKS.get(token)
    if not amazon_url:
        return "Invalid or expired link", 404

    # Get next keyword in rotation
    keyword = next(keyword_cycle)
    # Build Google search URL for that keyword
    google_url = "https://www.google.com/search?q=" + urllib.parse.quote_plus(keyword)

    # Render page that shows Google search, then auto-redirects to Amazon product page
    return render_template_string(
        REDIRECT_PAGE_TEMPLATE,
        keyword=keyword,
        google_url=google_url,
        amazon_url=amazon_url,
    )

if __name__ == "__main__":
    app.run(debug=True)
