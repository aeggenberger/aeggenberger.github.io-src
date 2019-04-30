Title: Coping With Twitter Follow Limits
Date: 2018-7-9 23:00
Category: API
Tags: twitter, python, api
Slug: coping-with-twitter-follow-limits

If you're like me, you run into [follow limits](https://help.twitter.com/en/using-twitter/twitter-follow-limit) on Twitter. 5,000 (as of this writing) isn't that many people, you think. And yet, Twitter is powerless to do anything about it.

> Follow limits cannot be lifted by Twitter and everyone is subject to limits, even high profile and API accounts.

"That's bullshit! It's not that they cannot; they just do not want to! Oh god, I need help!" you might be thinking. You're probably right. But if you're not ready for counseling and, like me, just want to be able to mash that follow button at will, I've discovered a temporary solution. 

This guide assumes you know how to code, ideally using [Python](https://www.python.org/). The first step is to download a client for the [Twitter Developer API](https://developer.twitter.com). I used [python-twitter](https://python-twitter.readthedocs.io/en/latest/index.html).

`$ pip install python-twitter`

Before you can start querying the Twitter API, you need authentication credentials. The python-twitter documentation has an easy-to-follow [guide](https://python-twitter.readthedocs.io/en/latest/getting_started.html). Completing the guide will leave you with your very own Twitter API object.

``` python
import twitter
api = twitter.Api(consumer_key=[consumer key],
                  consumer_secret=[consumer secret],
                  access_token_key=[access token],
                  access_token_secret=[access token secret],
				  sleep_on_rate_limit=True)
```
(Note: I've added a keyword argument to the object above relative to what appears in the guide. `sleep_on_rate_limit=True` will keep you and your 5,000 follows from running afoul of twitters API rate limits. It also means that some of the following commands will take a long time to run.)

Okay. Let's begin culling the list of people we follow. If you've been using Twitter a long time and follow thousands of people, there's a good chance some of the people you follow have stopped using the service without deleting their account. You are still following them, but they aren't contributing to your feed. This is the lowest hanging fruit.

Start by retrieving a list of all the accounts you follow:

``` python
following = api.GetFriends()
```

Using a little helper function to ignore Twitter data errors, we'll create a list of lists of the most recent tweet from each account.

``` python

def SafeGetUserTimeline(*args, **kwargs):
    try:
        return api.GetUserTimeline(*args, **kwargs)
    except twitter.TwitterError:
        return []

latest_tweets_lists = [SafeGetUserTimeline(user_id=friend.id, count=1)
                       for friend in following]
```

Now we flatten the list from a list of lists of tweets to just a list of tweets.

``` python

# flatten the lists
latest_tweets = [tweet for tweet_list in latest_tweets_lists
                 for tweet in tweet_list]
```

Next we sort the tweets by post date, and then use the sorted list to create a list of screen_names. The earlier in the list the screen name appears, the longer its been since that account tweeted. When I did this for my account, the longest-dormant account last tweeted in AD 2011.

``` python

def tweet_created_key(tweet):
    return tweet.created_at_in_seconds

# sort by creation time
latest_tweets.sort(key=tweet_created_key)

# list of screen names
screen_names = [tweet.user.screen_name for tweet in latest_tweets]

```

From this point on it's important to be sure that the commands you issue will do what you intend them to do. Don't execute anything you don't understand. It is possible to undo most unfollows just by re-following, but some accounts may have switched over to "protected" mode since you began following them, meaning the user will have to approve your request to follow them once again.

Ready to do this? Neat. To figure out which accounts I wanted to unfollow, I just began slicing the list from the beginning and checking to see if there were any accounts I still want to follow from that list. Eventually I settled on a list of 100 names.

``` python
names_to_unfollow = screen_names[:100]
```

If there is an account that you would like to continue following, just use list operations to remove it from `names_to_unfollow`. Now to do the actual unfollowing, we can use the `DestroyFriendship()` method of our api object.

``` python
for name in names_to_delete:
    api.DestroyFriendship(name)
```

The interpreter will print a line with information about each destroyed friendship.

Another source of candidates for unfollowing is this list of accounts that either don't have any tweets (either because they deleted them all or just haven't tweeted) or that triggered Twitter Data error while running SafeGetUserTimeline.

``` python
zero_tweets_or_error = [x.screen_name for x in following if len(SafeGetUserTimeline(user_id=x.id, count=1)) == 0]
```

You can amend and delete this list in the same way as before.

So there you have it: two ways to give yourself permission to follow again. Now that you can start following people again, how about following [this account](https://twitter.com/eggandburger)? Not! Just kidding! Or am I. Yes.
