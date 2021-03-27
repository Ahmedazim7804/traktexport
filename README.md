# traktexport

Export your Movies, TV shows and ratings from [`Trakt.tv`](https://trakt.tv/)

This isn't meant to be used to re-import info back into Trakt or export to another site, its just meant to save all my data so I have it locally, and can do analysis or graph my history.

## Installation

Requires `python3.6+`

To install with pip, run:

    pip install 'git+https://github.com/seanbreckenridge/traktexport'

## Usage

This uses OAuth to authenticate with the Trakt API (which afaik requires you to be a [VIP](https://trakt.tv/vip) on Trakt), see [here](https://pytrakt.readthedocs.io/en/latest/getstarted.html#oauth-auth) for more info.

This requires a manual setup the first time you use it, after which credentials are stored and this can run without any interaction.

### Setup

- Go to https://trakt.tv/oauth/applications and create a new application
- Use `urn:ietf:wg:oauth:2.0:oob` for the Redirect URI
- Run `traktexport auth yourtraktusername`
- Follow the instructions, pasting in your Client ID/Secret from the Trakt dashboard, going to the link and pasting the generated pin back into the terminal

Once you've done that, this saves OAuth refresh info in `${XDG_DATA_HOME:-$HOME/.local/share}/traktexport.json` (can overwrite location with the `TRAKTEXPORT_CFG` environment variable)

### Export

Then, to export all your ratings/movies/shows, run:

`traktexport export yourtraktusername > dump.json`

The results are printed to STDOUT, so `> dump.json` saves it to `dump.json`

```
$ python3 -m traktexport export yourTraktUsername > data.json
[D 210326 18:42:43 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/followers'...
[D 210326 18:42:45 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/following'...
[D 210326 18:42:48 export:32] Requesting 'https://api-v2launch.trakt.tv/users/settings'...
[D 210326 18:42:51 export:32] Requesting 'https://api-v2launch.trakt.tv/users/likes'...
[D 210326 18:42:54 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername'...
[D 210326 18:42:56 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/comments'...
[D 210326 18:42:59 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/lists'...
[D 210326 18:43:01 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/ratings'...
[D 210326 18:43:05 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/recommendations'...
[D 210326 18:43:07 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/watchlist'...
[D 210326 18:43:10 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/watched/movies'...
[D 210326 18:43:13 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/watched/shows'...
[D 210326 18:43:21 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/collection/movies'...
[D 210326 18:43:23 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/collection/shows'...
[D 210326 18:43:26 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/stats'...
[D 210326 18:43:29 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/history?limit=100&page=1'...
[D 210326 18:43:31 export:44] First item: {'id': 7353545729, 'watched_at': '2021-03-22T06:33:24.000Z', 'action': 'watch', 'type': 'movie', 'movie': {'title': 'Rain Man', 'year': 1988, 'ids': {'trakt': 304, 'slug': 'rain-man-1988', 'imdb': 'tt0095953', 'tmdb': 380}}}
[D 210326 18:43:31 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/history?limit=100&page=2'...
[D 210326 18:43:34 export:44] First item: {'id': 7178301624, 'watched_at': '2021-01-23T04:25:15.000Z', 'action': 'watch', 'type': 'episode', 'episode': {'season': 7, 'number': 7, 'title': 'Dangerous Debt', 'ids': {'trakt': 2590748, 'tvdb': 7640571, 'imdb': 'tt9313956', 'tmdb': 2201892, 'tvrage': None}}, 'show': {'title': 'Star Wars: The Clone Wars', 'year': 2008, 'ids': {'trakt': 4170, 'slug': 'star-wars-the-clone-wars', 'tvdb': 83268, 'imdb': 'tt0458290', 'tmdb': 4194, 'tvrage': 19187}}}
[D 210326 18:43:34 export:32] Requesting 'https://api-v2launch.trakt.tv/users/yourTraktUsername/history?limit=100&page=3'...
```

### Inspect

[`traktexport.dal`](./traktexport/dal.py) includes some code I use to parse the resulting JSON file into Python objects so its easier to manipulate

```python
class TraktExport(NamedTuple):
    username: str
    followers: List[Follow]
    following: List[Follow]
    likes: List[Like]
    stats: Dict[str, Any]
    settings: Dict[str, Any]
    watchlist: List[WatchListEntry]
    ratings: List[Rating]
    history: List[HistoryEntry]
```

```
python3 -m traktexport inspect data.json
Use 'data' to interact with the parsed TraktExport object

In [1]: data.history[0]
Out[1]: HistoryEntry(history_id=7353545729, watched_at=datetime.datetime(2021, 3, 22, 13, 33, 24, tzinfo=datetime.timezone.utc), action='watch', media_type='movie', media_data=Movie(title='Rain Man', year=1988, ids=SiteIds(trakt_id=304, trakt_slug='rain-man-1988', imdb_id='tt0095953', tmdb_id=380, tvdb_id=None, tvrage_id=None)))

In [2]: len(data.history)
Out[2]: 16063

In [3]: data.stats["movies"]["plays"]
Out[3]: 1511
```

Note: This does include this info the export, but it doesn't currently parse:

- collection
- comments
- lists
- recommendations

... because I don't have any of those on trakt

Created to use as part of [`HPI`](https://github.com/seanbreckenridge/HPI)
