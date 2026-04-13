# BDD

Given I am on Movie-Explorer main page  
When I enter a movie title and press enter  
Or when I enter a movie title and click search  
Then Matching results are displayed.  

## Implementation Steps

1. Define the search UX contract.
    - Add a text input for movie title and a Search button on the main page.
    - Trigger the same search action on Enter key press and Search button click.
    - Block empty submissions and show a short validation message.

2. Configure OMDb API access.
    - For production, keep the OMDb API key server-side only (BFF/API proxy or serverless function).
    - Frontend should call an internal endpoint (for example: `/api/movies?title=<title>&page=<page>`), not OMDb directly.
    - Proxy builds OMDb requests using `https://www.omdbapi.com/?apikey=<key>&s=<title>&type=<optional>&y=<optional>&page=<optional>&r=json`.
    - For local development only, allow environment variables in `.env.local` and never commit secrets.
    - URL-encode query values.

3. Implement API client logic.
    - Create a small API utility that accepts `title`, optional `type`, optional `year`, and optional `page`.
    - Handle OMDb response shape (`Search`, `totalResults`, `Response`, `Error`).
    - Return normalized data for the UI (`movies`, `totalResults`, `error`).

4. Manage UI states.
    - Add states for `idle`, `loading`, `success`, and `error`.
    - Show a loading indicator while requests are in progress.
    - Render movie cards/list when results are returned.
    - Show empty state when OMDb returns `Response: "False"` with `Error: "Movie not found!"`.

5. Support result pagination.
    - Keep current page in state and request next/previous pages with OMDb `page` parameter.
    - Disable pagination controls when at first page or when no additional pages are available.

6. Add optional detail lookup (recommended).
    - When a user selects a movie from the search list, call OMDb with `i=<imdbID>`.
    - Request `plot=full` for richer details in a detail panel or route.

7. Handle reliability and edge cases.
    - Trim input before search.
    - Guard against rapid repeated searches (disable button during loading or debounce input).
    - Handle network/API errors with a user-friendly message and retry option.

8. Verify against BDD behavior.
    - Confirm Enter key search returns matching results.
    - Confirm Search button click returns matching results.
    - Confirm invalid/empty queries do not send unnecessary requests.
    - Confirm error and empty states are rendered clearly.

9. Prepare deployment and request-limit resilience.
    - Add per-user request throttling on the proxy endpoint.
    - Add a short TTL cache on the proxy for repeated queries (for example: 60-300 seconds).
    - If OMDb returns limit/rate errors (or HTTP 429), return a friendly UI message and pause retries.
    - Use bounded retries with exponential backoff for transient failures only (network/5xx), not for invalid queries.
    - Log proxy errors with request context (query, page, timestamp, status/error) for monitoring.

10. Optional: generate API client with Kiota.
        - Create and maintain a minimal OpenAPI document for the OMDb operations used by this app (`s` search and `i` detail lookup), because OMDb does not provide an official OpenAPI spec.
        - Use Kiota to generate a typed client from that OpenAPI document.
        - Integrate the generated client in the backend proxy/serverless layer only, not in the browser.
        - Map generated response models into the app's normalized result shape (`movies`, `totalResults`, `error`).
        - Regenerate the client whenever the OpenAPI contract changes and validate behavior with smoke tests.

### Kiota Notes

- Why use Kiota:
    - Strongly typed request and response models.
    - Cleaner API integration boundaries in the proxy layer.
    - Easier long-term maintenance when request parameters evolve.

- Constraints:
    - OpenAPI accuracy is critical; generated client quality depends on the spec.
    - Kiota generation does not replace runtime protections like throttling, caching, and retry rules.
    - API key handling remains server-side regardless of client generation approach.

## Notes:

### External API Details
https://www.omdbapi.com/

Usage
Send all data requests to:
 > http://www.omdbapi.com/?apikey=[yourkey]&

Parameters
By ID or Title

| Parameter | Required  | Valid Options          | Default Value | Description                            |
|-----------|-----------|------------------------|---------------|----------------------------------------|
| i         | Optional* |                        | \<empty>      | A valid IMDb ID (e.g. tt1285016)       |
| t         | Optional* |                        | \<empty>      | Movie title to search for.             |
| type      | No        | movie, series, episode | \<empty>      | Type of result to return.              |
| y         | No        |                        | \<empty>      | Year of release.                       |
| plot      | No        | short, full            | short         | Return short or full plot.             |
| r         | No        | json, xml              | json          | The data type to return.               |
| callback  | No        |                        | \<empty>      | JSONP callback name.                   |
| v         | No        |                        | 1             | API version (reserved for future use). |

*\*Please note while both "i" and "t" are optional at least one argument is required.*

By Search

| Parameter | Required | Valid Options          | Default Value | Description                            |
|-----------|----------|------------------------|---------------|----------------------------------------|
| s         | Yes      |                        | \<empty>      | Movie title to search for.             |
| type      | No       | movie, series, episode | \<empty>      | Type of result to return.              |
| y         | No       |                        | \<empty>      | Year of release.                       |
| r         | No       | json, xml              | json          | The data type to return.               |
| page      | No       | 1-100                  | 1             | Page number to return.                 |
| callback  | No       |                        | \<empty>      | JSONP callback name.                   |
| v         | No       |                        | 1             | API version (reserved for future use). |


```mermaid

sequenceDiagram
    Movie-Lookup->>Open Movie DB API: Request movies by title
    Open Movie DB API-->>Movie-Lookup: Movies List