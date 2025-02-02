---
published: true
title: A simple login manager for your Flask API
layout: post
---
Here's a simple implementation of the `login_required` function
for your flask api.

Firstly, we'd a means of authentication and I'd be using
`Authorization: Bearer Token` as my means of authorization and 
`JWT` (JSON web token) to encrypt certain data or in this case create access tokens.

{% highlight py %}
# -*- coding: utf-8 -*-
# login_manager.py

from flask import current_app
from datetime import datetime, timedelta
from functools import wraps
import jwt


class APILoginManager(object):

	def __init__(self, app=None):

		self.login_message = "You need to login"

		if app is not None:
			self.init_app(app)

	def init_app(self, app):
		"""
		Configures an application. This registers an `after_request` call, and
        attaches this `LoginManager` to it as `app.login_manager`.

		:param app: The :class:`flask.Flask` object to configure.
        :type app: :class:`flask.Flask`
		"""

		app.login_manager = self

	def user_loader(self, callback):
		'''
		This sets the callback for reloading a user. The
		function you set should take a user ID and return a
		user object, or ``None`` if the user does not exist.

		:param callback: The callback for retrieving a user object.
		:type callback: callable
		'''
		self._user_callback = callback
		return callback


	def encode_token(self, user_id, minutes):
		""" Generates a user access token

		:param minutes: The token's age 
		:type  minutes: integer

		:return: user access token
		"""

		# set up a payload with an expiration time
		payload = {
			'exp': datetime.utcnow() + timedelta(minutes=minutes),
			'iat': datetime.utcnow(),
			'user_id': user_id
		}

		# create the byte string token using the payload and the SECRET key
		jwt_string = jwt.encode(
			payload,
			current_app.config.get('SECRET_KEY'),
			algorithm='HS256'
		)

		if type(jwt_string) == bytes:
			jwt_string = jwt_string.decode()

		return jwt_string

	def decode_token(self, token):
		"""Decodes the token from the Authorization header.
			
			:param token: a valid access token
			:type  token: String

			:return: (token_is_valid, user_id)
		"""

		try:
			# try to decode the token using our SECRET variable
			payload = jwt.decode(token, current_app.config.get('SECRET_KEY'), algorithms=['HS256'])
			return True, payload['user_id']
		except jwt.ExpiredSignatureError:
			# the token is expired, return an error string
			return False, "Expired token. Please login to get a new token"
		except jwt.InvalidTokenError:
			# the token is invalid, return an error string
			return False, "Invalid token. Please register or login"

		return False, "Invalid token. Please register or login"

	def get_token_for(self, user, minutes=60):
		try:
			user_id = str(user.id)
		except AttributeError:
			raise NotImplemented("No `id` attribute")

		return self.encode_token(user_id, minutes)
{% endhighlight %}

The `decode_token` and `encode_token` property functions are simply used to **decode** end **encode**
payloads respectively using JWT (JSON Web Tokens). From the code you'd understand that we can set a age limit (when the token would expire) for an access token.

However, we'd need a decorator function to utilize the class above. Here would call
this function `login_required`.

{% highlight py %}
# utils.py

from flask import (
	jsonify, 
	request, 
	current_app
)
from functools import wraps

def login_required(func):

	@wraps(func)
	def decorator(*args, **kwargs):
		auth_header   = request.headers.get('Authorization')
		error_message = current_app.login_manager.login_message

		if auth_header:
			try:
				# Expected format (Bearer TOKEN)
				token = auth_header.split(" ")[1]
			except IndexError:
				return jsonify(
					dict(
						status=401, 
						message=error_message
					)
				), 401

			valid, user_id = current_app.login_manager.decode_token(token)

			if valid:
				user = current_app.login_manager._user_callback(user_id)
				
				return func(*args, **kwargs, current_user=user)
			else:
				# Note: if not valid, user_id holds an error message
				error_message = user_id

		return jsonify(
			dict(
				status=401, 
				message=error_message
			)
		), 401

	return decorator
{% endhighlight %}

[Here](https://github.com/keosariel/api-login-manager/blob/master/README.md) is an example to test our little [login manager](https://github.com/keosariel/api-login-manager).

[Full source code](https://github.com/keosariel/api-login-manager)
I hope this was helpful tho.
