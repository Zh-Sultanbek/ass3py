Team: Zhakeyev Sultanbek(SE-2017), Almas Askaruly(SE-2011)

Title: Homework
Installation: pip install PyJWT; pip install Flask; pip install Flask-SQLAlchemy
Usage:
import jwt
from flask import Flask, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///db.db'
db = SQLAlchemy(app)

class User(db.Model):
	__tablename__ = 'User'
	id = db.Column('id', db.Integer, primary_key=True)
	login = db.Column('login', db.Text, nullable=False)
	password = db.Column("password", db.Text, nullable=False)
	token = db.Column("token", db.Text, nullable=True)

@app.route("/login", methods=['GET'])
def login():
	login = request.args.get('login')
	password = request.args.get('password')
	if not login or not password:
		return "Type a login and a password both of them"
	else:
		user = User.query.filter_by(login=login).first()
		if user:
			if user.login == login:
				if user.password == password:
					encoded = jwt.encode({"token": str(user.id)}, "secret", algorithm="HS256")
					user.token = encoded
					db.session.commit() # update
					
					return f"token: {user.token}"
				else:
					return f"Could not found a user with login: {login}"
		else:
			return f"Could not found a user with login: {login}"

@app.route("/protected", methods=['GET'])
def protected():
	token = request.args.get('token')
	user = User.query.filter_by(token=token).first()
	if (user):
		return "<h1>Hello, token which is provided is correct </h1>"
	else:
		return "<h1>Hello, Could not verify the token </h1>"		

if __name__ == "__main__": # if the code starts in the console
	app.run(debug=True)
