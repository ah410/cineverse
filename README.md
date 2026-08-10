# CineVerse

Find information on popular movies that are currently out. Browse trending
titles, search by name, and watch trailers, all in one place.

**Demo:** [YouTube Showcase](https://youtu.be/FrbTbngjF1g)

## Branches

- **`master`** (default) runs on PostgreSQL via Flask-SQLAlchemy. This is the
  actively developed implementation, deployed at
  [cineversehd.com](https://www.cineversehd.com). It doesn't currently have a
  working automated test suite: the checked-in `api/app_test.py` predates the
  move to SQLAlchemy and targets an older SQL API, so it won't run correctly
  against `master` as-is.
- **`sqlite`** is an earlier implementation on SQLite (via CS50's SQL library)
  with a working `pytest`/`unittest` suite. Check out this branch if you want to
  run the tests locally.

## Setup

Instructions below target Windows + VS Code; steps are equivalent on macOS/Linux
aside from the virtual-environment activation command and PowerShell-specific
notes.

1. Clone the repository:
   ```bash
   git clone https://github.com/ah410/cineverse.git
   cd cineverse
   ```
2. Create and activate a virtual environment (requires Python 3.11.x, 3.12+ is
   not supported):
   ```bash
   python -m venv myenv
   myenv/scripts/activate
   ```
   If Windows blocks the activation script, run PowerShell as Administrator and
   execute `Set-ExecutionPolicy RemoteSigned` first.
3. Install dependencies (includes `pytest`):
   ```bash
   pip install -r requirements.txt
   ```
4. Set the required environment variables (see below).
5. Run the app:
   ```bash
   cd api
   flask run
   ```
6. Run tests (on the `sqlite` branch only, see Branches above):
   ```bash
   pytest app_test.py
   # or
   python -m unittest app_test.py
   ```

### Environment variables

| Variable | Purpose |
|---|---|
| `TMDB_API_KEY` | The Movie Database API key, used to fetch movie listings/details. |
| `YouTube_API_KEY` | YouTube Data API v3 key, used to fetch trailer links. Without it, the movie description page (trailer view) won't load; login and home page still work. Get a free key via the [Google Cloud Console](https://console.cloud.google.com/) and the [Google API Python Client docs](https://github.com/googleapis/google-api-python-client/blob/main/docs/start.md). |
| `SECRET_KEY` | Flask session secret. |
| `POSTGRESQL_URL` | Postgres connection string (`master` branch only). |

On Windows, set these under User Variables. See
[these instructions](https://phoenixnap.com/kb/windows-set-environment-variable).

## Features

**Login/Sign-Up.** Users land on the login page by default; unauthenticated
requests to any route redirect there. Sign-up takes a username and password
(confirmed twice), storing a `werkzeug.security` password hash rather than the
plaintext password.

**Home Page.** A grid of movie posters and titles pulled from TMDB, rendered via
a Jinja loop. Hovering enlarges the poster; clicking a movie loads its
description page.

**Search.** A search bar on the homepage matches movie titles using a SQL
`LIKE` query with wildcards against the locally cached movie table.

**Description Page.** Shows the enlarged poster, title, release date,
description, and an embedded trailer. The trailer is resolved at request time
via a YouTube Data API v3 search for `"<title> trailer"`, extracting the first
result's video ID for the embed URL.

## Technology Used

**Front-End:** HTML, CSS, JavaScript, Bootstrap

**Back-End:** Python, Flask

**Data:** PostgreSQL (`master`) / SQLite via CS50 SQL (`sqlite` branch), The
Movie Database (TMDB) API, YouTube Data API v3

## Technical Highlights

- Login-required redirect: a `login_required` decorator (using `*args`/
  `**kwargs`) wraps protected routes and redirects unauthenticated requests to
  `/login`.
- TMDB and YouTube API integration: movie listings come from TMDB's popular
  movies endpoint, and trailers are resolved via the YouTube Data API v3
  (`googleapiclient.discovery.build`), searching by title and extracting the
  video ID from the first search result to build an embeddable IFrame URL.
  Scraping approaches (Selenium, BeautifulSoup) were tried first but abandoned
  since YouTube's search results are client-side rendered.
- Homepage layout: movie cards are rendered via a Jinja loop with a fixed
  percentage width for a fixed grid; each poster is wrapped in a form that
  POSTs the clicked movie's ID (via a hidden input) to load its description
  page.

## Future Improvements

1. Favorite movies: let users save movies to a favorites list.
2. Advanced search: filter by title, genre, rating, or release year.
3. User reviews and ratings.
4. Sort options on the homepage.

## References

1. [TMDB API Reference](https://developer.themoviedb.org/reference/movie-popular-list)
2. [TMDB poster_path base URL discussion](https://www.themoviedb.org/talk/568e3711c3a36858fc002384)
3. [google-api-python-client](https://github.com/googleapis/google-api-python-client/tree/main)
4. [YouTube Data API v3 docs](https://developers.google.com/youtube/v3/docs/)
5. [YouTube IFrame Player docs](https://developers.google.com/youtube/player_parameters/#Manual_IFrame_Embeds)
6. [Bootstrap navbar tutorial](https://www.youtube.com/watch?v=qNifU_aQRio)
7. [Bootstrap spacing classes](https://mdbootstrap.com/docs/standard/utilities/spacing/)
8. [Bootstrap color classes](https://mdbootstrap.com/docs/standard/content-styles/colors/)
9. [Login Required tutorial](https://www.youtube.com/watch?v=CD5lFKyH9Ls)
10. [Python YouTube API tutorial](https://www.youtube.com/watch?v=th5_9woFJmk)
