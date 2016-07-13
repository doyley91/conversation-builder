# Conversation builder for Tweet data
## Fiona Pigott

Python script to assemble individual Tweets from a public Twitter stream (either Gnip activity-streams format or original Twitter API format) into conversation threads, using the "in reply to" fields of the tweet payload.

Input: Stream of JSON-formatted Tweet payloads, one record per line.
Input (optional): A CSV of brand handles and user ids, one handle/id pair per line, no quotes. E.g.: notFromShrek,2382763597) 
Output: Stream of JSON-formatted *conversation* payloads, one record per line. Ordering in the output will be pretty much random.

# Output format:

A few notes about the output:
* User screen names are all converted to lower case. The only case where there are capital letters in the "screen_name field and "
* Tweets can be "missing" if a Tweet appears in an "in reply to" field, but not in the input dataset. In this case, we take the Tweet ID, user screen name, and the user id (user id of the user being replied to is only available in the original format). If Tweets are unavalible in the dataset, some fields will be undefined.
* All of the output fields that are brand-related will not appear if no --brand_info is provided
* The only different between activity-streams and original format output is that the "tweets" field will contain Tweet paylaods formatted in the same way as the input.

Output might look like this, when my provided --brand_info was:
delta,5920532
deltaassist,137460929

`
{
  "nonbrands_tweeting": [{"screen_name": < screen name 1 >,"user_id": < user id 1 >}, {"screen_name": < screen name 2 >,"user_id": < user id 2 >}],
  "nonbrands_mentioned": [{"screen_name": < screen name 1 >,"user_id": < user id 1 >}], # users who are mentioned in the thread, but are not brands
  "time_to_first_response": "00:57:38", # H:M:S that it took for the first person to respond to this Tweet 
  "brands_mentioned": [{"screen_name": delta, "user_id": "5920532}], # Brands that were mentioned
  "ids_of_missing_tweets": [ ... ], # this is a Tweet that we know was part of the conversation, but whose data was not in the input. empty if all Tweets were in the input
  "size_of_conversation": 2, # number of Tweets in the conversation
  "tweets": [
    { < Tweet payload > }, # if the first Tweet was missing, it is an empy dictionary in the payload
    { < Tweet payload > }
  ],
  "first_brand_response": {
   < payload of the Tweet that was the brand's first response in this thread > 
  },
  "brands_tweeting": [{"screen_name": "deltaassist","user_id": "137460929"}],
  "time_to_first_brand_response": "00:57:38", # H:M:S that it took the brand to respond to this Tweet
  "root_user": {"screen_name": < screen name 1 >,"user_id": < user id 1 >},
  "approx_depth": 1, # Depth of the reply chain
  "duration_of_conversation": "00:57:38" # (time of the last Tweet) - (time of the first Tweet)
}
`

# Usage
`
cat raw_Tweet_data.json | python build_conversations.py --brand_info csv_of_brand_Twitter_handles_and_ids.csv --format activity-streams --log build_conversations.log > conversation_threads.json 
`

# Requirements:

* Python 3
* ujson (Python package for fast JSON encoding and decoding)
* MongoDB
* PyMongo (Python package for interfacing with MongoBD)

# Setup

## MongoDB
On MacOSX, I was able to install MongoDB with the homebrew package manager (https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/). I also had to create a /data/db directoy, with a user:group fiona:staff.
On a server running Ubuntu 14.04, I installed MongoDB following these instuctions (https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/) and it worked right away.

## Python 3
If you don't have Python 3, install that first. Among other things, Python 3's default UTF-8 character encoding is important for this usecase.
Then:
`pip install ujson`
`pip install pymongo`
(If you're not working in a PYthon virtual environment, you may have to `sudo pip install`)
