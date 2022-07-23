```python
!pip install requests
!pip install beautifulsoup4
```


```python
import base64
```


```python
import requests
from bs4 import BeautifulSoup
import re

import json
import pandas as pd
```


```python
client_id = "66855b632e234127a5a3f325a6965229"
client_secret = "9c55406a1b0d496d91570c386538b1df"
```


```python
client_creds = f"{client_id}:{client_secret}"
```


```python
client_creds_b64 = base64.b64encode(client_creds.encode())
```


```python
token_url = "https://accounts.spotify.com/api/token"
method = "POST"
token_data = {
    "grant_type": "client_credentials"
}
token_headers = {
    "Authorization": f"Basic {client_creds_b64.decode()}"
}

token_headers
```


```python
r = requests.post(token_url, data = token_data, headers = token_headers)
test = r.json()
token = test["access_token"]
print(token)
```


```python
uri_list = []
song_list = []
danceability = []
tempo_list = []
artist_list = []
energy = []
valence = []
```


```python
url = "https://www.billboard.com/pro/summer-songs-1985-present-top-10-tunes-each-summer-listen/"

result = requests.get(url)
doc = BeautifulSoup(result.text, "html.parser")

#pull all pargraphs from html
data = []
for para in doc.find_all("p"):
    data.append(para.get_text())
    #print(para.get_text())

#rid unnecessary paragraphs from data pull
del data[:5], data[64:]

#format and compile data into dictionary
position = 0
compilation = {}
for i in range(len(data)):   
    commas=', (.*?),'   
    a = re.findall(commas, data[position])
    del a[0]
    
    compilation[data[position][:4]] = a
    position += 1
```


```python
#the other dude's code
data = []
for p in doc.select('.pmc-paywall p:has(strong)'):
    for s in [dict(zip(p.find_all(text=True)[1].split(','),s.strip().split(', '))) for s in p.find_all(text=True)[2:]]:
        #print(s)
        s.update({'year':p.strong.text})
        data.append(s)
df = pd.DataFrame(data)

df.columns = ['Rank', 'Title', 'Artist', 'Year']

artist_list = df["Artist"].tolist()
song_list = df["Title"].tolist()
year_list = df["Year"].tolist()

newsonglist = []
for song in song_list:
    if "/" in song:
        newsonglist.append(song[:song.find("/")])
    elif "— " in song:
        newsonglist.append(song.replace("— ", ""))
    else:
        s1 = re.sub("[()–]", "", song)
        newsonglist.append(s1)
    
song_list = newsonglist
print(artist_list[150])
```


```python
#edit lists for query formatting

finallist = []
for song, artist in zip(song_list, artist_list):
    finallist.append(f"{song} - {artist}")
```


```python
errors = []
apisong_list = []
apiartist_list = []
class GetSummerSongs:
    def __init__(self):
        self.id = client_id
        self.secret = client_secret
    
    def get_songs(self):
        
        for song in (finallist):
            try:
                query = ("https://api.spotify.com/v1/search?q={}&type=track&limit=4").format(song)
                response = requests.get(query,
                                        headers={"Content-Type": "application/json", "Authorization": "Bearer {}".format(token)})
                response_json = response.json()
                
                apisong_list.append(response_json["tracks"]["items"][0]["name"])
                apiartist_list.append(response_json["tracks"]["items"][0]["artists"][0]["name"])
                uri_list.append(response_json["tracks"]["items"][0]["id"])
            except:
                print("ERROR")
                errors.append(song)


a = GetSummerSongs()
a.get_songs()

print("finished")
```


```python
print(errors)
```


```python
original = pd.DataFrame({"OG Songs": song_list,
                           "OG Artists": artist_list,})
```


```python
new = pd.DataFrame({"API Songs": apisong_list,
                    "API Artists": apiartist_list,})
```


```python
#export dataframe at given listing
#new.to_csv('new', index = False)
```


```python
query = ("https://api.spotify.com/v1/search?q=KISS%20ME%20MORE%20-%20Doja%20Cat&type=artist,track")
response = requests.get(query,
                        headers={"Content-Type": "application/json", "Authorization": "Bearer {}".format(token)})

response_json = response.json()
print(response_json)
```


```python
testlist = ['ONLY THE LONELY (KNOW HOW I FEEL)', 'WALK — DON’T RUN', 'IT’S TOO LATE/I FEEL THE EARTH MOVE', 'GIVE ME LOVE – (GIVE ME PEACE ON EARTH)', 'NEVER LEAVE YOU – UH OOH']
newtestlist = []
for song in errors:
    if "/" in song:
        newtestlist.append(song[:song.find("/")])
    elif "— " in song:
        newtestlist.append(song.replace("— ", ""))
    else:
        s1 = re.sub("[()–]", "", song)
        newtestlist.append(s1)
        
print(errors)
print(newtestlist)
```
