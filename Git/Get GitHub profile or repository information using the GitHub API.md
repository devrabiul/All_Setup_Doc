Get GitHub profile or repository information using the GitHub API. GitHub provides a public REST API and a GraphQL API for fetching user and repo details.

### 1️⃣ Get User Profile Information
Use the following endpoint to get a GitHub user’s public profile:

```bash
GET https://api.github.com/users/{username}
```

#### Example:
```bash
curl -s https://api.github.com/users/octocat
```

#### Response:
```json
{
  "login": "octocat",
  "id": 583231,
  "avatar_url": "https://avatars.githubusercontent.com/u/583231?v=4",
  "html_url": "https://github.com/octocat",
  "public_repos": 8,
  "followers": 10000,
  "following": 5
}
```

### 2️⃣ Get User Repositories
```bash
GET https://api.github.com/users/{username}/repos
```

#### Example:
```bash
curl -s https://api.github.com/users/octocat/repos
```

### 3️⃣ Get Repository Information
```bash
GET https://api.github.com/repos/{owner}/{repo}
```

#### Example:
```bash
curl -s https://api.github.com/repos/octocat/Hello-World
```

### Authentication (For Rate Limits & Private Repos)
GitHub API has rate limits for unauthenticated requests. If you exceed the limit, use a GitHub Personal Access Token.

#### Example with Token:
```bash
curl -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
     -s https://api.github.com/users/octocat
```
