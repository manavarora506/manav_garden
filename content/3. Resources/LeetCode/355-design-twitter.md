# 355. Design Twitter

**Date:** 2026-02-02
**Difficulty:** Medium
**Topics:** Heap, Design, Hash Map
**Link:** https://leetcode.com/problems/design-twitter/

> ⚠️ **TODO:** Practice this problem again - had many bugs during implementation.

## Final Solution

```python
class Twitter:

    def __init__(self):
        self.user_to_following = defaultdict(list)
        self.counter = 0
        self.user_to_tweet = defaultdict(list)

        
    def postTweet(self, userId: int, tweetId: int) -> None:
        self.counter += 1
        self.user_to_tweet[userId].append((self.counter, tweetId))

    def getNewsFeed(self, userId: int) -> List[int]:
        following = self.user_to_following.get(userId, []) + [userId]
        max_heap = []
        news_feed = []
        for followee in following:
            user_tweets = self.user_to_tweet[followee]
            if user_tweets != []:
                idx = len(user_tweets) - 1
                ts, last_tweet = user_tweets[idx]
                heapq.heappush(max_heap, (-ts, last_tweet, followee, idx))

        while max_heap and len(news_feed) < 10:
            ts, last_tweet, followee, index = heapq.heappop(max_heap)
            news_feed.append(last_tweet)
            curr_user_tweets = self.user_to_tweet[followee]
            index -= 1
            if index >= 0:
                next_ts, next_tweet = curr_user_tweets[index]
                heapq.heappush(max_heap, (-next_ts, next_tweet, followee, index))
        
        return news_feed

    def follow(self, followerId: int, followeeId: int) -> None:
        if followeeId not in self.user_to_following[followerId]:
            self.user_to_following[followerId].append(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        if followeeId in self.user_to_following[followerId]:
            self.user_to_following[followerId].remove(followeeId)
```

## Initial Issues (Many!)

1. **heapify returns None**: Tried `max_heap = heapq.heapify(...)` 

2. **Wrong assignment**: `self.user_to_tweet = []` instead of `self.user_to_tweet[userId] = []`

3. **Can't concatenate list + int**: `following + userId` instead of `following + [userId]`

4. **Destroying original data**: Used `.pop()` on tweet list instead of using index

5. **Wrong loop condition**: `while heap or len < 10` instead of `while heap and len < 10`

6. **Inconsistent variable names**: `min_heap` vs `max_heap`

7. **Using old timestamp**: Pushed old `ts` instead of new tweet's timestamp

8. **Wrong indexing**: `user_tweets[-1][idx]` double-indexed instead of `user_tweets[idx]`

9. **Undefined variables**: `index` instead of `idx`, `next_latest_tweet` instead of `next_tweet`

10. **Duplicate follows**: Could add same followee twice without checking

## Key Learnings

### This is Merge K Sorted Lists

Each user has their own sorted list of tweets (by timestamp). `getNewsFeed` merges tweets from user + all followees to get the 10 most recent.

### Data Structures Needed

```python
user_to_following: {userId: [followeeIds]}  # Who does this user follow
user_to_tweet: {userId: [(ts, tweetId), ...]}  # User's tweets in chronological order
counter: int  # Global timestamp for ordering tweets
```

### Heap Entry for Merge

```python
(-timestamp, tweetId, userId, index)
#    ^          ^        ^       ^
#    |          |        |       +-- Where to get next tweet from this user
#    |          |        +---------- Which user this tweet belongs to
#    |          +------------------- The tweet to add to feed
#    +------------------------------ Negated for max heap (most recent first)
```

### Don't Modify Original Data

Use an index pointer instead of popping from the original list:
```python
# WRONG - destroys data
user_tweets.pop()

# RIGHT - use index
idx = len(user_tweets) - 1
ts, tweet = user_tweets[idx]
idx -= 1  # Move to next tweet
```

## Nuances to Remember

- Check for duplicate follows before adding
- Check if followee exists before removing (unfollow)
- Use `defaultdict` to avoid key existence checks
- Include the user themselves in the feed (not just who they follow)
- Store index in heap entry to know where to get next tweet

## Q&A Highlights

**Q:** Should we precompute feeds or compute on-demand?
**A:** On-demand is better. Precomputing means updating all followers' feeds on every post, and rebuilding on follow/unfollow.

**Q:** Why is this merge k sorted lists?
**A:** Each user has a sorted list of tweets. getNewsFeed merges lists from user + followees, taking the 10 most recent overall.
