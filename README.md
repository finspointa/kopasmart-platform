# KopaSmart API

API service for KopaSmart


## Installation(For Development)

`sudo apt install postgresql`

`sudo apt install postgresql-server-dev-10`

`pip install -r requirements.txt`

Create postgres database

Go to `settings.py` and change database name, user and password accordingly

Make migrations `python manager.py makemigrations`

Migrate `python manager.py migrate`


## Running(In a Developer Mode)

`python manage.py runserver`

To access the API go to http://localhost:8000/


## API Documentation
The base route for the version in production is `https://api.kopasmart.app/`

### Available HTTP Methods & Their Standard Usage
* **GET:** For listing/retrieving resource(s)
* **POST:** For creating a resource
* **PUT:** For updating a resource fully(Send data with all fields required during resource creation)
* **PATCH:** For updating a resource partially(Send data with only fields which you want to update)
* **DELETE:** For deleting a resource

### Available Routes
* **register:** [`/register/`](#Register-User)
* **login:** [`/auth/`](#Authenticate-User)
* **users:** [`/users/`](#Users)
* **user-follows-list:** [`/users/{your_user_id}/follows/`](#to-get-a-list-of-all-users-you-follows-use)
* **user-following-list:** [`/users/{your_user_id}/following/`](#to-get-a-list-of-all-users-following-you-use)
* **profile-pictures:** [`/profile-pictures/`](#Profile-Pictures)
* **lender-profiles:** [`/lender-profiles/`](#Lender-Profiles)
* **lender-profile-ratings:** [`/lender-profile-ratings/`](#Lender-Profile-Ratings)
* **borrower-profiles:** [`/borrower-profiles/`](#Borrower-Profiles)
* **posts:** [`/posts/`](#Posts)
* **post-medias:** [`/post-medias/`](#Post-Medias)
* **comments:** [`/comments/`](#Comments)
* **user-posts:** [`/posts/{user_id}/posts/`](#to-get-a-list-of-all-users-postsexcluding-comments)
* **user-comments:** [`/comments/{user_id}/comments/`](#to-get-a-list-of-all-users-comments)
* **user-feeds:** [`/feeds/`](#to-get-users-feeda-list-of-posts-from-users-i-follow-sorted-by-date)
* **repayment-reschedules:** [`/repayment-reschedules/`](#Repayment-Schedules)

### Register User
Available Routes
* `/register/`

Available HTTP Method
* POST

Data Format
```js
{
    "username": "String",
    "email": "valid email",
    "password": "password",
    "role": "borrower/lender/admin"  // Defaults to borrower if you don't specify
}
```

### Authenticate User
Available Routes
* `/auth/`

Available HTTP Methods
* POST

Data Format
```js
{
    "username": "valid username",
    "password": "valid password"
}
```

### Users 
Available Routes
* `/users/`
* `/users/{user_id}/`

Available HTTP Methods
* GET
* PUT
* PATCH

Data Format
```js
{
    "first_name": "String",
    "middle_name": "String",
    "last_name": "String",
    "username": "valid username",
    "email": "valid email",
    "phone": "valid phone number",
    "biography": "Text",
    "picture": "ProfilePicture",
    "lender_profile": "LenderProfile",
    "borrower_profile": "BorrowerProfile",
    "follows": "User"  // This is a write only field
    "follows_count": "Integer",  // This gives a total number of users you follows
    "following_count": "Integer", // This gives a total number of users following you

    // Tells if the retrieved user is following you(must be authenticate for this to work)
    "is_following_me": "Boolean",
    "date_joined": "DateTime(String)",
    "updated_at": "DateTime(String)",
    "role": "borrower/lender/admin",
    "is_admin": "Boolean",
    "is_borrower": "Boolean",
    "is_lender": "Boolean"
    "is_active": "Boolean"
}
```

#### To follow users use
POST `/follow-user/`

Request body
```js
{
    // This tlanslates to add this user to a list of users who I follow
    // Must be authenticated for this to work
    "id": id_of_user_to_follow
}
```

#### To unfollow users use
POST `/unfollow-user/`

Request body
```js
{
    // This tlanslates to remove this user from a list of users who I follow
    // Must be authenticated for this to work
    "id": id_of_user_to_unfollow
}
```

#### To get a list of all users you follows use
GET `/users/{your_user_id}/follows/`

#### To get a list of all users following you use
GET `/users/{your_user_id}/following/`

#### To search a user
GET `/users/?search=your_key_word`

This searches the keyword in the following fields, `username`, `first_name`, `middle_name` and `last_name`

### Profile Pictures
Available Routes
* `/profile-pictures/`
* `/profile-pictures/{profile_picture_id}/`

Available HTTP Methods
* GET
* POST
* PUT
* PATCH
* DELETE

Data Format
```js
{
    "owner": "String",  // Read only
    "src": "Image(string)"
}
```

### Lender Profiles
Available Routes
* `/lender-profiles/`
* `/lender-profiles/{lender_profile_id}/`

Available HTTP Methods
* GET
* PUT
* PATCH

Data Format
```js
{
    "raters_count": "Integer",  // This is a total number of raters(Read only)
    "total_rating_score": "Float",  // This is a calculated rating score(Read only)
    "user": "User(Integer)"  // Id of user who owns this profile(Read only)
}
```
**NOTE:** There is no need to create this manually, that's why it doesn't support `POST`. It is usually created automatically when signing up with a `lender` role or setting the value of user's `is_lender` to `True`. If you set the value of user's `is_lender` to `False` the lender profile associated with it will be deleted.

### Lender Profile Ratings
Available Routes
* `/lender-profile-ratings/`
* `/lender-profile-ratings/{lender_profile_ratings_id}/`

Available HTTP Methods
* GET
* POST
* PUT
* PATCH
* DELETE

Data Format
```js
{
    "score": "Integer(0 - 5)",  // This is rater's score, ranges from 0 to 5
    "rater": "User(Integer)",  // This is rater's id(Read only)
    "profile": "LenderProfile(Integer)"  // Id of lender's profile
}
```

**NOTE:** You can't rate lender's profile more than once. `profile` field supports write with `POST` https method only, In other words you can't edit it once you have have sent `POST`.

#### To rate lender's profile use
POST `/lender-profile-ratings/`


Request body
```js
{
    "profile": 7,  // Here 7 is the ID of lenders profile I want to rate
    "score": 4  // Here 4 is my rating score 
}
```

**NOTE:** You must be authenticated for this to work(that's the reason why you don't need to provide rater's id)

### Borrower Profiles
Available Routes
* `/borrower-profiles/`
* `/borrower-profiles/{borrower_profile_id}/`

Available HTTP Methods
* GET
* PUT
* PATCH

Data Format
```js
{
    "user": "User(Integer)"  // Id of user who owns this profile(Read only)
    // No more custom fields for now, but there will be more in the future
}
```
**NOTE:** There is no need to create this manually too, that's why it doesn't support `POST` as well, this is usually created automatically when signing up with a `borrower` role or setting the value of user's `is_borrower` to `True`. If you set the value of user's `is_borrower` to `False` the borrower profile associated with it will be deleted.

### Posts
Available Routes
* `/posts/`
* `/posts/{post_id}/`

Available HTTP Methods
* GET
* POST
* PUT
* PATCH
* DELETE

Data Format
```js
{
    "owner": "User",  // Read only

    // A post can either be of type `post`(top parent) or type `comment`(which is a comment to another post/comment)
    "type": "post/comment",  // Read only(Defaults to post when you create a post)
    "text_content": "Text",
    "media_type": "picture/video"
    "medias": "List(PostMedia)",
    "likes_count": "Integer",  // Number of likes(Read only)

    // Tells if I have liked the retrieved post(must be authenticate for this to work)
    "is_liked_by_me": "Boolean",
    "comments_count": "Integer",  // Number of comments(Read only)
    "created_at": "DateTime(string)",  // Read only
    "updated_at": "DateTime(string)",  // Read only
}
```

### Post Medias
Available Routes
* `/post-medias/`
* `/post-medias/{post_media_id}/`

Available HTTP Methods
* GET
* POST
* PUT
* PATCH
* DELETE

Data Format
```js
{
    "post": "Post(Integer)",  // Post id
    "src": "File(String)",  // A file(either a picture or a video)
}
```

### Comments(Child of Post)
This has all fields as `Post` plus a `post` field which store its parent. Comments can have childs(sub comments) too.

Available Routes
* `/comments/`
* `/comments/{comment_id}/`

Available HTTP Methods
* GET
* POST
* PUT
* PATCH
* DELETE

Data Format
```js
{
    "owner": "User",  // Read only
    "post": "Post(Integer)",  // Parent post's id
    "type": "post/comment",  // Read only(Defaults to comment when you create a comment)
    "text_content": "Text",
    "media_type": "picture/video"
    "medias": "List(PostMedia)",
    "likes_count": "Integer",  // Number of likes(Read only)

    // Tells if I have liked the retrieved post(must be authenticate for this to work)
    "is_liked_by_me": "Boolean",
    "comments_count": "Integer",  // Number of comments(Read only)
    "created_at": "DateTime(string)",  // Read only
    "updated_at": "DateTime(string)",  // Read only
}
```

To create a simple post(with `type=post`) with texts only you only need to send one `POST` request, E.g

POST `/posts/`

Request body
```js
{
    "text_content": "Hello there, this is my first post"
    // The rest fields are optional
}
```

To create a simple comment(with `type=comment`) with texts only you only need to send one `POST` request as well, E.g

POST `/comment/`

Request body
```js
{
    "post": 1,  // The ID of post you want to comment/reply to
    "text_content": "Hello there, this is my first comment"
    // The rest fields are optional
}
```

To create a simple comment(with `type=comment`) to another comment with texts only you only need to send one `POST` request as well, E.g

POST `/comments/`

Request body
```js
{
    "post": 3,  // The ID of comment you want to comment/reply to
    "text_content": "Hello there, this is my first reply to comment"
    // The rest fields are optional
}
```

To create a post/comment with media(s) only or media(s) with texts you need to send atleas two `POST` request as well, one for creating a post/comment and other `n` requests for creating `n` number medias, E.g

POST `/posts/`

Request body
```js
{
    "text_content": "Hello there, this is my first post with a media",
    "media_type": "picture"
}
```

Let's say this creates a post with `id=1`, Now send another `POST` request to create a post media

POST `/post-medias/`

Request body
```js
{
    "post": 1,
    "src": "Attach your media file here(picture/video)"
}
```

You can create as many medias as you want as long as they belong to the post/comment which you own.

**NOTE:** You Must be authenticated to create a post/comment/post-media.


#### Now how to like/unlike a post/comment ?
Here is how to like

POST `/like-post/`

Request body
```js
{
    // This tlanslates to add this post to a list of posts I like
    // Must be authenticated for this to work
    "id": id_of_post_to_like
}
```

To unlike(undo like) the post/comment

POST `/unlike-post/`

Request body
```js
{
    // This tlanslates to remove this post from a list of posts I like
    // Must be authenticated for this to work
    "id": id_of_post_to_unlike
}
```

**NOTE:** You must be authenticated to do this

#### To get a list of post's comments
GET `/posts/{post_id}/comments/`

#### To get a list of comment's comments
GET `/comments/{comment_id}/comments/`

#### To get a list of all user's posts
GET `/users/{user_id}/posts/`

#### To get a list of all user's posts(excluding comments)
GET `/users/{user_id}/posts/?type=post`

#### To get a list of all user's comments
GET `/users/{user_id}/comments/`

#### To get user's feed(a list of posts from users I follow sorted by date)
GET `/feeds/`
**NOTE:** You must be authenticated to do this

### Repayment Schedules
Available Routes
* `/repayment-schedules/`
* `/repayment-schedules/{schedule_id}/`

Available HTTP Methods
* GET
* POST
* PUT
* PATCH

Data Format
```js
{
    "name": "Repayment Schedule name",
    "amount": "Float",
    "interest_rate": "Float",
    "interest_method": "FLAT/STRAIGHT/REDUCING_BALANCE",
    "loan_term": "Integer",
    "loan_term_unit": "D/W/M/Y",
    "repaid_every": "Integer",
    "repaid_every_unit": "D/W/M/Y",
    "number_of_repayments": "Integer",
    "amortization": "EQUAL_INSTALLMENT/EQUAL_PRINCIPAL",
    "interest_moratorium": "Integer",
    "principal_moratorium": "Integer",
    "interest_free_period": "Integer"
}
```
