---
title: "Bad passwords and the NIST guidelines"
date: 2020-07-05
tags: [Python, Data Science, Cybersecurity]
header:
  image: "/images/passwordnist/main_image.jpg"
excerpt: "Checking what passwords conform to the National Institute of Standard and Technology guidelines."
---
> Written by Devanshu Ramaiya and Manan Jhaveri 

## Introduction
In this blogpost, we will go through the rules in [NIST Special Publication 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html) and check what a *verifier* (what the NIST calls a second party responsible for storing and verifying passwords) should perform to make sure users don't pick bad passwords. We will go through the passwords of users from a fictional company and use python to flag the users with bad passwords.

*Warning: The list of passwords and the fictional user database both contain real passwords leaked from real websites. These passwords have not been filtered in any way and include words that are explicit, derogatory and offensive.*

Let's start by importing pandas and the relevant dataset. We will also take a look at the number of users we have and the first 12 users.
```python
  # Importing the pandas module
  import pandas as pd

  # Loading in datasets/users.csv
  users = pd.read_csv('datasets/users.csv')

  # Printing out how many users we've got
  print(len(users))

  # Taking a look at the 12 first users
  print(users.head(12))
```
<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/users.PNG" alt="First 12 users in the database">

## Passwords should not be too short
What do you think is the first thing we should be checking according to the NIST Special Publication 800-63B?

> Verifiers SHALL require subscriber-chosen memorized secrets to be at least 8 characters in length.

Okay, so the passwords of our users shouldn't be too short. Let's check that out!

```python
  # Calculating the lengths of users' passwords
  users['length'] = users['password'].str.len()

  # Flagging the users with too short passwords
  users['too_short'] = (users['length'] < 8)

  # Counting and printing the number of users with too short passwords
  print("Number of too short passwords:", users['too_short'].value_counts()[1])

  # Taking a look at the 12 first rows
  print(users.head(12))
```
Here's the output -
<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/tooshort.PNG" alt="Users with too short passwords">

## Common passwords people use
Next up in Special Publication 800-63B is the rule that
> Verifiers SHALL compare the prospective secrets against a list that contains values known to be commonly-used, expected or compromised.
- Passwords obtained from previous breach corpuses.
- Dictionary words.
- Repetitive or sequential characters (e.g. 'aaaa', '1234abcd').
- Context-specific words, such as the name of the service, the username, and derivatives thereof.

We are gonna check these in order and start with *Passwords obtained from previous breach corpuses*, that is, websites where hackers have leaked all the user's passwords. As many websites don't follow NIST guidelines and encrypt passwords there now exist large lists of the most popular passwords. Let's start by loading in the 10,000 most common passwords which have been taken from [here](https://github.com/danielmiessler/SecLists/tree/master/Passwords).

```python
  # Reading in the top 10000 passwords
  common_passwords = pd.read_csv('datasets/10_million_password_list_top_10000.txt', header=None, squeeze=True)

  # Taking a look at the top 20
  print(common_passwords.head(20))
```
Take a look at the top 20 common passwords in the list and I hope yours is not one of them😉

<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/top20_common.PNG" alt="Top 20 common passwords">

Now, let's flag all the passwords in our user database that are among the top 10,000 used passwords.

```python
  # Flagging the users with passwords that are common passwords
  users['common_password'] = (users['password'].isin(common_passwords))

  # Counting and printing the number of users using common passwords
  print("Number of users with common passwords: ", users['common_password'].value_counts()[1])

  # Taking a look at the 12 first rows
  print(users.head(12))
```
Let's look at the number of users using common passwords and the first 12 rows in the users database.
<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/common_password.PNG" alt="Number of people using common passwords">
Wow, we already have 129 out of 982 users who use common passwords!

## Passwords should not be common words
Next rule states that -
> Verifiers SHALL compare the prospective secrets against a list that contains [...] dictionary words.

It is easy for hackers to check users' passwords against common English words and therefore common English words make bad passwords. Let's check our users' passwords against the top 10,000 English words from [Google's Trillion Word Corpus](https://github.com/first20hours/google-10000-english).
```python
  # Reading in a list of the 10000 most common words
  words = pd.read_csv('datasets/google-10000-english.txt', header=None, squeeze=True)

  # Flagging the users with passwords that are common words
  users['common_word'] = (users['password'].str.lower()).isin(words)

  # Counting and printing the number of users using common words as passwords
  print("Number of users using common words as passwords:", users['common_word'].value_counts()[1])

  # Taking a look at the 12 first rows
  print(users.head(12))
````
So, what do you think? What will be the number of users using common words as passwords?
<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/common_word.PNG" alt="Number of people using common words as passwords">

## Passwords should not be your name
> Verifiers SHALL compare the prospective secrets against a list that contains [...] context-specific words, such as the name of the service, the username, and derivatives thereof.

One thing to notice that in the user's database the username consists of first name and last name separated by a dot. For now, let's just flag passwords that are the same as either a user's first or last name.

```python
  # Extracting first and last names into their own columns
  users['first_name'] = users['user_name'].str.extract(r'(^\w+)', expand=False)
  users['last_name'] = users['user_name'].str.extract(r'(\w+$)', expand=False)

  # Flagging the users with passwords that matches their names
  users['uses_name'] = (((users['password'].str.lower()) == users['first_name']) | ((users['password'].str.lower()) == users['last_name']))

  # Counting and printing the number of users using names as passwords
  print("Number of users using names as passwords:", users['uses_name'].value_counts()[1])

  # Taking a look at the 12 first rows
  print(users.head(12))
```
Here's the output -
<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/uses_name.PNG" alt="Number of people using their first or last names as passwords">

## Passwords should not be repetitive
To check for repetitiveness can be arbitrarily complex, but here we are only going to do something simple. We're going to flag all passwords that contain 4 or more repeated characters.

```python
### Flagging the users with passwords with >= 4 repeats
users['too_many_repeats'] = users['password'].str.contains(r'(.)\1\1\1')
```
The number of users with too many repeats is 6.

## Combining everything
Now we have implemented all the basic tests for bad passwords suggested by NIST Special Publication 800-63B! What's left is just to flag all bad passwords and strongly suggest these users to change their password.
```python
  # Flagging all passwords that are bad
  users['bad_password'] = (users['too_short'] | users['common_password'] | users['common_word'] | users['uses_name'] | users['too_many_repeats'])

  # Counting and printing the number of bad passwords
  print("Number of bad passwords:", users['bad_password'].value_counts()[1])

  # Looking at the first 25 bad passwords
  print(users[users['bad_password'] == True]['password'].head(25))
```
What do you think, how many users have bad passwords? And what are those bad passwords? Let's take a look at the number of people with bad passwords and the first 25 bad passwords.
<img src="{{ site.url }}{{ site.baseurl }}/images/passwordnist/bad_passwords.PNG" alt="Number of people having bad passwords">

424 out of 982 users in the database have bad passwords, that's about 42% of the users!!!

## Otherwise, the password should be up to the user
We have implemented the password checks recommended by NIST Special Publication 800-63B. It's certainly possible to better implement these checks, for example by using a longer list of common passwords. Also, note that the NIST checks in no way guarantee that a chosen password is good, just that it's not obviously bad.

Apart from the checks implemented above, NIST is also clear with what password rules should not be imposed:
> Verifiers SHOULD NOT impose other composition rules (e.g., requiring mixtures of different character types or prohibiting consecutively repeated characters) for memorized secrets. Verifiers SHOULD NOT require memorized secrets to be changed arbitrarily (e.g., periodically)

So, the next time a website or an app tells you to "include both a number, symbol and an upper and lower case character in your password" you should send them a copy of [NIST Special Publication 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html).

## Credits
I would like to thank [DataCamp](https://www.datacamp.com/) for making such amazing content and giving us the opportunity to do a project like this.
