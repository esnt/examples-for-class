---
layout: default
---

# Example 4
*Note: All identifying links have been removed for anonymity*

---

#### **Introduction**
This is Part 2 of my project exploring BYU MBA Slack data. Now that our data is cleaned up and prepared for EDA, we will dive in and attempt to answer a few questions that will, hopefully, uncover some interesting insights about communication patterns and informal social networks in the BYU MBA program. Those questions are:
- Who reacts to messages most?
- What are the most popular emojis?
- Who generates the most reactions (total and per message)?
- Who replies to messages most?
- Who generates the most replies (total and per message)?
- Who generates the fastest replies?
- What kind of networks exist within channel membership?

#### **Question 1: Who reacts to messages most?**

To answer this question (see code below) we will grab a subset of our main data frame that only includes the 'user' and 'reactions' columns. Each record in the 'reactions' column contains a string of nested JSON, so we will use a function with a regular expression to extract the 'count' value (the number of reactions) into a new column. We will then create another function that will split things out of that column in a coherent way into a dictionary of lists. Then we simply take that list of dictionaries and concatenate them together into a single data frame and merge that with the 'users' data frame to get the real names of people. With clean, isolated data on reactions, we can now make a bar graph with that data frame that shows us the top ten reactors.

```python
import re
import ast
from pandas import json_normalize
import matplotlib.pyplot as plt

react1 = pd.DataFrame(data[data['reactions'].notna()][['user','reactions']])

def num_and_sum(s):
    pattern = r"'count': (\d+)"
    counts = [int(num) for num in re.findall(pattern, s)]
    return sum(counts)

react1['sum_counts'] = react1['reactions'].astype(str).apply(num_and_sum)
react1a = react1.groupby('user')['sum_counts'].sum().reset_index()

def convert_string_to_dict(s):
    data = ast.literal_eval(s)
    result = {'name': [], 'users': [], 'count': []}
    
    for item in data:
        result['name'].extend([item['name']] * item['count'])
        result['users'].extend(item['users'])
        result['count'].extend([item['count']] * item['count'])
    
    return result

list_of_dic = react1['reactions'].astype(str).apply(convert_string_to_dict).tolist()

# Concatenate all data frames into one
react2 = pd.DataFrame({'name': [], 'users': [], 'count': []})
for i in list_of_dic:
    max_length = max(len(lst) for lst in i.values())
    for key in i:
        i[key].extend([None] * (max_length - len(i[key])))
    df = pd.DataFrame(i)
    react2 = pd.concat([react2, df], ignore_index=True)

# Join users table to get real names for users
react2b = pd.merge(react2, users[['id', 'real_name']], left_on='users', right_on = 'id', how='left')
react2b

# Visualize
react2a = react2b[['real_name', 'count']].groupby('real_name').count().reset_index().sort_values(by='count', ascending=False)
paired = sorted(zip(react2a['count'][0:10], react2a['real_name'][0:10]))
count_s, users_s = zip(*paired)
plt.barh(users_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value-12, index, str(value), va='center')
plt.xlabel('Messages Reacted To')
plt.ylabel('Name')
plt.title('Top Ten Reactors (Oct 2023)')
plt.show()
```
<html>
<!-- <style>
.center {display: block; margin-left: auto; margin-right: auto;
  width: 50%; /* Adjust as needed */
}
</style> -->
    <img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topreactors.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

#### **Question 2: What are the most popular emojis?**

We can answer now more questions about reactions like "What are the most popular emojis?". We will use the same 'react2' data frame that we just created. The top ten are dominated by positive reactions. We can also see that the trend here is less linear and more curved, which suggests that one person's emoji use may influence other people's emoji use.

```python
react2c = react2[['name', 'count']].groupby('name').count().reset_index().sort_values(by='count', ascending=False)
paired = sorted(zip(react2c['count'][0:10], react2c['name'][0:10]))
count_s, emoji_s = zip(*paired)
plt.barh(emoji_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value-55, index, str(value), va='center')
#plt.xlabel('Count')
plt.ylabel('Emoji')
plt.title('Top Ten Emojis (Oct 2023)')
plt.show()
```
<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topemojis.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

#### **Question 3: Who generates the most reactions (total and per message)?**

Let's go a bit deeper and ask what is (perhaps) a more meaningful question: "Who _generates_ the most reactions? It is one thing to count reactions but reaction _generators_ may function as a proxy for finding those that have influence, or at least those who are creating valuable content. For this, we will go back to our 'react1a' data frame to get info on the user that posted messages and the 'react_count' column that Slack has conveniently provided (so we don't have to extract numbers again from the JSON string in the 'reactions' column. 

We will be careful though and make sure that we don't just look at total reactions but also reactions per message since we could get a distorted picture with those who have a very small number of messages that happened to have a lot of reactions. This second part will require a bit more manipulation from our original data frame with all message data to get the denominator of total messages for people. If we wanted to go further (which I don't in this case) we could filter out those who have less than 10 messages to preserve the quality of this metric.

```python
#Top Reaction Generators by Total Reactions
react1b = pd.merge(react1a, users[['id', 'real_name']], left_on='user', right_on = 'id', how='left')
react1b

react1c = react1b.sort_values(by='react_count', ascending=False)
paired = sorted(zip(react1c['react_count'][0:10], react1c['real_name'][0:10]))
count_s, user_s = zip(*paired)
plt.barh(user_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value-35, index, str(value), va='center')
#plt.xlabel('Count')
plt.ylabel('Name')
plt.title('Top Ten Reaction Generators - Total (Oct 2023)')
plt.show()
```
<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topreactgen.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

```python
# Top Reaction Generators by Reactions per Message
import numpy as np

mess1 = all0[['user','ts']].groupby('user').count().reset_index().sort_values(by='ts', ascending=False).dropna()
mess1.rename(columns={'ts': 'mess_count'}, inplace=True)
react1a.rename(columns={'sum_counts': 'react_count'}, inplace=True)
mess1a = pd.merge(pd.merge(react1a, mess1, on='user', how='left'), users[['id', 'real_name']], left_on='user', right_on = 'id', how='left')
mess1a['re_per_mes'] = round(mess1a['react_count']/mess1a['mess_count'],2)
mess1a = mess1a[['real_name', 're_per_mes']]
mess1a['real_name'] = mess1a['real_name'].astype(str)

mess1b = mess1a.sort_values(by='re_per_mes', ascending=False)
paired = sorted(zip(mess1b['re_per_mes'][0:10], mess1b['real_name'][0:10]))
count_s, user_s = zip(*paired)
print(count_s, user_s)
plt.barh(user_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value-2, index, str(value), va='center')
#plt.xlabel('Count')
plt.ylabel('Name')
plt.title('Top Ten Reaction Generators - Reactions Per Message (Oct 2023)')
plt.show()
```

<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topreactgen2.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

#### **Question 4: Who replies to messages most?**

Another metric we can look at to measure Slack activity is replies. Who replies to messages most? Whether these people are more helpful than others, more extroverted, or just spend more time on Slack won't be clear, but it may provide an interesting point of comparison to those with the most reactions. I hypothesize that this would be a similar group of people seeing how both are comparable dimensions of slack activity. Fortunately, Slack also provides a data point on each message called the 'parent_user_id'. This is only on those messages that are replies (in a thread) and it identifies the user who posted the original message that the replier is responding to - which we will leverage to uncover this metric.

```python
reply = all0[['user', 'parent_user_id']].dropna().groupby('user').count().reset_index().sort_values(by='parent_user_id', ascending=False)
reply.rename(columns={'parent_user_id': 'reply_count'}, inplace=True)
reply1 = pd.merge(reply, users[['id', 'real_name']], left_on='user', right_on = 'id', how='left')

paired = sorted(zip(reply1['reply_count'][0:20], reply1['real_name'][0:20]))
count_s, user_s = zip(*paired)
print(count_s, user_s)
plt.barh(user_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value-2.4, index, str(value), va='center')
plt.ylabel('Name')
plt.title('Top Ten Repliers (Oct 2023)')
plt.show()
```
<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topreply.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

#### **Question 5: Who generates the most replies (total and per message)?**

As we did with reactions, we can go a bit deeper and ask 'Who _generates_ the most replies?". Similarly, these may be people who have more influence or valuable content. We will take a similar approach with the code. 

```python
reply2 = all0[['parent_user_id', 'ts']].dropna().groupby('parent_user_id').count().reset_index().sort_values(by='ts', ascending=False)
reply2.rename(columns={'parent_user_id': 'user', 'ts': 'replies_gen'}, inplace=True)
reply2a = pd.merge(reply2, users[['id', 'real_name']], left_on='user', right_on = 'id', how='left')
print(reply2a)

paired = sorted(zip(reply2a['replies_gen'][0:20], reply2a['real_name'][0:20]))
count_s, user_s = zip(*paired)
print(count_s, user_s)
plt.barh(user_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value-3.4, index, str(value), va='center')
plt.ylabel('Name')
plt.title('Top Ten Reply Generators (Oct 2023)')
plt.show()
```
<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topreplygen.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

#### **Question 6: Who generates the fastest replies?**

Going a step further, another measure of influence or value related to reply generation would be _how fast_ replies are generated. For this, we take the timestamp on each reply, subtract the timestamp of the original message, and then group each of those times by user and take the average. Once again, if we were engaged in a more serious review of this data, it would be appropriate to filter out those who generated less than 10 replies.

```python
reply3 = all0[['parent_user_id', 'thread_ts', 'ts']].dropna()
reply3['reply_time'] = (reply3['thread_ts'] - reply3['ts'])/60*-1
reply3 = reply3.groupby('parent_user_id')['reply_time'].mean().reset_index().sort_values(by='reply_time', ascending=True)
reply3.rename(columns={'parent_user_id': 'user', 'reply_time': 'avg_reply_time'}, inplace=True)
reply3a = pd.merge(reply3, users[['id', 'real_name']], left_on='user', right_on = 'id', how='left').dropna()
reply3a['real_name'] = reply3a['real_name'].astype(str)
reply3a['avg_reply_time'] = round(reply3a['avg_reply_time'], 2)
print(reply3a)

paired = sorted(zip(reply3a['avg_reply_time'][0:20], reply3a['real_name'][0:20]), reverse=True)
count_s, user_s = zip(*paired)
print(count_s, user_s)
plt.barh(user_s, count_s)
for index, value in enumerate(count_s):
    plt.text(value, index, str(value), va='center')
plt.xlabel('Avg Reply Time in Minutes')
plt.ylabel('Name')
plt.title('Top Ten Fastest Reply Generators (Oct 2023)')
plt.show()
```
<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/topfastreply.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:500px;"/><br><br></html>

#### **Question 7: What kind of networks exist within channel membership?**

To take a look at how the network structure looks between messages and replies, we can use the networkx package, but first we will have to prepare a data frame that includes the names (joined from the users data frame) of each message-poster and message-replier. Then we'll add nodes and edges with a loop, select the kamada_kawai_layout (among other options), and graph it.

```python
# Create message-reply data frame with names
replynet = all0[['user', 'parent_user_id']].dropna().reset_index()[['user', 'parent_user_id']].drop_duplicates()
replynet = replynet[replynet['user'] != replynet['parent_user_id']]
replynet = pd.merge(replynet, users[['id', 'real_name']], left_on='user', right_on = 'id', how='left').dropna()
replynet.rename(columns={'real_name': 'reply_name'}, inplace=True)

replynet = pd.merge(replynet.drop('id', axis=1), users[['id', 'real_name']], left_on='parent_user_id', right_on = 'id', how='left').dropna()
replynet.rename(columns={'real_name': 'parent_name'}, inplace=True)
replynet['reply_name'] = replynet['reply_name'].astype(str)
replynet['parent_name'] = replynet['parent_name'].astype(str)
replynet = replynet[['parent_name','reply_name']]

# Create network graph
import networkx as nx
G = nx.Graph()

## Add nodes and edges
for _, row in replynet.iterrows():
    user, parent_user = row['user'], row['parent_user_id']
    G.add_node(user)
    G.add_node(parent_user)
    G.add_edge(user, parent_user)

## Draw the graph
pos = nx.kamada_kawai_layout(G)#, k=1, iterations=40)  # You can experiment with the k value and iterations
nx.draw(G, pos, node_size=75, font_size=3, font_color="black", node_color="blue", alpha=1)
label_pos = {key:[value[0], value[1]-0.05] for key, value in pos.items()}  # Adjust label positions
nx.draw_networkx_labels(G, label_pos, font_size=3, font_color="black")
plt.show()
```
<html><img src="https://raw.githubusercontent.com/bwegr/386/main/assets/images/network2.png" alt="" style="display: block; margin-left: auto; margin-right: auto; width:1000px;"/><br><br></html>

#### **Conclusion**
We've only scratched the surface on what we can do with Slack data, but we've uncovered a few interesting insights in the process. Unsurprisingly, out of 200 students, we see the same group of 10-20 students in the densest part of the network graph (Hiwa Nae'ole, Kelsee Gates, Matt Benson, Bette Benson, Brittany Bennion, Benjamin Williams, Megan Lesa, Allison Adams, Ashlee Love, etc.) that we see in many of the previous graphs and measures on Slack activity (replies, reactions, etc.). While we can't necessarily say just by looking at activity that these specific people have more influence or value compared to others, this is a great starting towards uncovering more solid insights as Slack is a major part of BYU MBA communication and, at the very least, this data shows us a digital shadow of the informal, social connections in the program.


