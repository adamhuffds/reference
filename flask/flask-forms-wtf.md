# Flask Reference — Forms & User Input (Flask-WTF)

Docs: https://flask-wtf.readthedocs.io/

## Why Flask-WTF Over Raw `request.form`

Reading `request.form.get(...)` directly (see `flask-basics.md`) works for
simple cases, but doesn't give you validation, error messages, or CSRF
protection. Flask-WTF (built on the WTForms library) is the standard
solution once a form needs more than one or two fields, or handles anything
security-sensitive like login/signup.

```bash
pip install flask-wtf
```

## CSRF Protection Setup

```python
app.config["SECRET_KEY"] = "change-this-in-production"   # required — used to sign CSRF tokens
```
Flask-WTF automatically protects every form against CSRF (Cross-Site Request
Forgery) attacks as long as `SECRET_KEY` is set and the token is included in
the template (see below). This is on by default and is one of the main
reasons to prefer Flask-WTF over hand-rolled forms.

## Defining a Form Class

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField, IntegerField
from wtforms.validators import DataRequired, Email, Length, NumberRange

class SignupForm(FlaskForm):
    username = StringField("Username", validators=[DataRequired(), Length(min=3, max=20)])
    email = StringField("Email", validators=[DataRequired(), Email()])
    password = PasswordField("Password", validators=[DataRequired(), Length(min=8)])
    age = IntegerField("Age", validators=[NumberRange(min=0, max=120)])
    submit = SubmitField("Sign Up")
```
Common field types: `StringField`, `PasswordField`, `IntegerField`,
`BooleanField`, `SelectField`, `TextAreaField`, `FileField`.
Common validators: `DataRequired`, `Email`, `Length`, `NumberRange`,
`EqualTo` (e.g. confirm-password fields), `Optional`.
Docs: https://wtforms.readthedocs.io/en/3.1.x/fields/ | https://wtforms.readthedocs.io/en/3.1.x/validators/

## Handling the Form in a View

```python
@app.route("/signup", methods=["GET", "POST"])
def signup():
    form = SignupForm()
    if form.validate_on_submit():          # True only on POST + all validators pass
        username = form.username.data
        email = form.email.data
        # ... save to DB ...
        return redirect(url_for("home"))
    return render_template("signup.html", form=form)
```
`validate_on_submit()` combines checking the request method is POST *and*
running all field validators — the standard one-line check for "was this
form submitted correctly."

## Rendering the Form in a Template

```html
<form method="POST">
    {{ form.hidden_tag() }}        <!-- renders the CSRF token, required -->

    {{ form.username.label }} {{ form.username() }}
    {% for error in form.username.errors %}
        <span class="error">{{ error }}</span>
    {% endfor %}

    {{ form.email.label }} {{ form.email() }}
    {{ form.password.label }} {{ form.password() }}

    {{ form.submit() }}
</form>
```
`form.hidden_tag()` is what actually inserts the CSRF token field — easy to
forget, and without it every submission will fail with a 400 error.

## File Uploads

```python
from flask_wtf.file import FileField, FileAllowed, FileRequired
import os
from werkzeug.utils import secure_filename

class UploadForm(FlaskForm):
    file = FileField("Upload File", validators=[
        FileRequired(),
        FileAllowed(["csv", "xlsx"], "CSV and Excel files only")
    ])

@app.route("/upload", methods=["GET", "POST"])
def upload():
    form = UploadForm()
    if form.validate_on_submit():
        f = form.file.data
        filename = secure_filename(f.filename)      # sanitize filename, strips path traversal attempts
        f.save(os.path.join("uploads", filename))
        return "Uploaded!"
    return render_template("upload.html", form=form)
```
`secure_filename()` is important any time you save a user-uploaded filename
to disk — without it, a malicious filename like `../../etc/passwd` could
attempt path traversal. Docs: https://werkzeug.palletsprojects.com/en/latest/utils/#werkzeug.utils.secure_filename

Raw file access without Flask-WTF (for simple cases):
```python
@app.route("/upload", methods=["POST"])
def upload_raw():
    f = request.files["file"]           # from an HTML <input type="file" name="file">
    f.save(os.path.join("uploads", secure_filename(f.filename)))
    return "Uploaded!"
```

## Custom Validators

```python
class SignupForm(FlaskForm):
    username = StringField("Username", validators=[DataRequired()])

    def validate_username(self, field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError("Username already taken")
```
Any method named `validate_<fieldname>` on the form class runs automatically
as an extra validator — the standard way to add custom logic (e.g. database
uniqueness checks) beyond WTForms' built-in validators.