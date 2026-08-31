from pathlib import Path
import zipfile

root = Path("/mnt/data/muhammad-arsalan-github-profile")
(root / ".github" / "workflows").mkdir(parents=True, exist_ok=True)
(root / "scripts").mkdir(parents=True, exist_ok=True)

readme = r'''# Muhammad Arsalan

**Software · Cloud · AI · Unity · XR · 3D**

Karachi, Pakistan · [LinkedIn](https://www.linkedin.com/in/muhammadarsalan111/) · [Email](mailto:YOUR_EMAIL)

## About

I build software, games, immersive applications, AI/data solutions, and 3D engineering products.

My work spans **SaaS, web and mobile applications, Unity/VR/AR, 2D and 3D games, Python/AI, cloud systems, industrial simulation, digital twins, and product design**.

---

## Projects

<!-- PROJECTS:START -->

| Project | Description | Tech | Status | Updated |
|---|---|---|---|---|
| Loading from GitHub... | — | — | — | — |

<!-- PROJECTS:END -->

> Project details, languages, stars, forks, links, and update dates are fetched automatically from GitHub.  
> **Status** is the only intentionally manual field, maintained in `scripts/project_status.json`.

---

## Currently Building

<!-- CURRENTLY_BUILDING:START -->

| Repository | Description | Activity | Last Updated |
|---|---|---|---|
| Loading from GitHub... | — | — | — |

<!-- CURRENTLY_BUILDING:END -->

Repositories are selected automatically from recent GitHub development activity.

---

## GitHub Activity

<!-- DAILY_ACTIVITY:START -->

| Time | Activity | Repository |
|---|---|---|
| Loading from GitHub... | — | — |

<!-- DAILY_ACTIVITY:END -->

The activity feed is generated from public GitHub events.

---

## GitHub Overview

<!-- GITHUB_STATS:START -->

| Metric | Value |
|---|---:|
| Public repositories | Updating |
| Contributions this year | Updating |
| Contributions last 30 days | Updating |
| Commits / pushes | Updating |
| Pull requests | Updating |
| Issues | Updating |
| Stars | Updating |
| Followers | Updating |
| Last activity | Updating |
| Last refresh | Updating |

<!-- GITHUB_STATS:END -->

---

## Tech Stack

| Area | Technologies |
|---|---|
| Languages | Python · JavaScript · TypeScript · C# · SQL |
| Web | React · Node.js · HTML · CSS |
| Mobile | React Native |
| Cloud | AWS |
| AI / Data | Python · Data Science · Machine Learning · AI |
| Games / XR | Unity · VR · AR · MR · 2D · 3D |
| 3D / Engineering | Fusion 360 · Rhinoceros · Gemvision Matrix · ZBrush |
| Database / Tools | PostgreSQL · Git · GitHub |

---

## Certifications

| Certification | Issuer | Credential ID | Validity |
|---|---|---|---|
| **PMP®** | PMI | `4420925` | Jun 2026 – Jun 2029 |
| **AWS Solutions Architect – Associate** | AWS | `b4cf1e3cb25a434391099348f7c7527b` | Jul 2026 – Jul 2029 |
| **AWS CloudOps Engineer – Associate** | AWS | `df8e221d0eac4dddbbf131951898fb20` | Jul 2026 – Jul 2029 |
| **AWS Cloud Practitioner** | AWS | `e2dadccb90364c6585d136dc71ab06ee` | Jul 2026 – Jul 2029 |
| **AI Using Python** | DigiSkills / Ignite | `PATWXUGMK` | Jul 2026 |

[Verify PMP®](https://www.pmi.org/certifications/certification-resources/registry) · [Verify AWS](https://aws.amazon.com/verification) · [Verify DigiSkills](https://digiskills.pk/verify)

---

## Areas of Interest

**AI & Data Science** · **Cloud & AWS** · **SaaS** · **Unity & Game Development** · **VR / AR / MR** · **Digital Twins** · **Mobile Applications** · **3D Engineering** · **Product Design** · **Automation**

---

## Connect

[LinkedIn](https://www.linkedin.com/in/muhammadarsalan111/) · [GitHub](https://github.com/MuhammadArsalaN1) · [Email](mailto:YOUR_EMAIL)

---

<sub>Project and activity data are automatically synchronized from GitHub.</sub>
'''

status = r'''{
  "Casino-VR": "Active",
  "HRMS": "Maintained",
  "khataBook": "Maintained",
  "Fusion360-Techincal-work": "Maintained",
  "ERP-Goat-Management-system-": "Dormant",
  "Linear-Cleaning-System": "Active"
}
'''

workflow = r'''name: Sync GitHub Profile README

on:
  schedule:
    - cron: "17 */6 * * *"
  workflow_dispatch:

permissions:
  contents: write

jobs:
  sync-profile:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Generate README data
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITHUB_USERNAME: MuhammadArsalaN1
        run: python scripts/update_readme.py

      - name: Commit README changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"

          if git diff --quiet -- README.md; then
            echo "README is already up to date."
            exit 0
          fi

          git add README.md
          git commit -m "chore: sync GitHub profile"
          git push
'''

script = r'''import json
import os
import re
from datetime import datetime, timezone, timedelta
from urllib.request import Request, urlopen

USERNAME = os.environ.get("GITHUB_USERNAME", "MuhammadArsalaN1")
TOKEN = os.environ["GITHUB_TOKEN"]
README_FILE = "README.md"
STATUS_FILE = "scripts/project_status.json"

API = "https://api.github.com"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "Accept": "application/vnd.github+json",
    "X-GitHub-Api-Version": "2022-11-28",
    "User-Agent": "github-profile-readme-sync",
}

def get_json(url):
    request = Request(url, headers=HEADERS)
    with urlopen(request, timeout=30) as response:
        return json.loads(response.read().decode("utf-8"))

def graphql(query, variables):
    payload = json.dumps({"query": query, "variables": variables}).encode()
    headers = {
        **HEADERS,
        "Content-Type": "application/json",
    }
    request = Request(
        "https://api.github.com/graphql",
        data=payload,
        headers=headers,
        method="POST",
    )
    with urlopen(request, timeout=30) as response:
        result = json.loads(response.read().decode("utf-8"))

    if "errors" in result:
        raise RuntimeError(result["errors"])

    return result["data"]

def replace_section(text, name, replacement):
    pattern = rf"(<!-- {re.escape(name)}:START -->).*?(<!-- {re.escape(name)}:END -->)"
    result, count = re.subn(
        pattern,
        lambda m: f"{m.group(1)}\n\n{replacement}\n\n{m.group(2)}",
        text,
        flags=re.S,
    )
    if count != 1:
        raise RuntimeError(f"Expected exactly one {name} section.")
    return result

def relative_time(value):
    if not value:
        return "—"

    date = datetime.fromisoformat(value.replace("Z", "+00:00"))
    delta = datetime.now(timezone.utc) - date
    seconds = int(delta.total_seconds())

    if seconds < 60:
        return "just now"
    if seconds < 3600:
        return f"{seconds // 60}m ago"
    if seconds < 86400:
        return f"{seconds // 3600}h ago"
    if seconds < 2592000:
        return f"{seconds // 86400}d ago"
    if seconds < 31536000:
        return f"{seconds // 2592000}mo ago"
    return f"{seconds // 31536000}y ago"

def table(headers, rows):
    output = [
        "| " + " | ".join(headers) + " |",
        "| " + " | ".join("---" for _ in headers) + " |",
    ]

    for row in rows:
        output.append("| " + " | ".join(str(x) for x in row) + " |")

    return "\n".join(output)

def event_description(event):
    event_type = event.get("type", "")
    payload = event.get("payload", {})

    if event_type == "PushEvent":
        count = len(payload.get("commits", []))
        return f"Pushed {count} commit{'s' if count != 1 else ''}"

    if event_type == "PullRequestEvent":
        number = payload.get("number", "")
        action = payload.get("action", "updated")
        return f"PR #{number} {action}"

    if event_type == "IssuesEvent":
        issue = payload.get("issue", {})
        number = issue.get("number", "")
        action = payload.get("action", "updated")
        return f"Issue #{number} {action}"

    if event_type == "CreateEvent":
        return f"Created {payload.get('ref_type', 'repository')}"

    if event_type == "DeleteEvent":
        return f"Deleted {payload.get('ref_type', 'branch')}"

    if event_type == "ReleaseEvent":
        return "Published release"

    if event_type == "ForkEvent":
        return "Forked repository"

    if event_type == "WatchEvent":
        return "Starred repository"

    return event_type.replace("Event", "") or "Activity"

# ---------------------------------------------------------------------
# GitHub account
# ---------------------------------------------------------------------

user = get_json(f"{API}/users/{USERNAME}")

# ---------------------------------------------------------------------
# Public repositories
# ---------------------------------------------------------------------

repositories = []
page = 1

while True:
    batch = get_json(
        f"{API}/users/{USERNAME}/repos"
        f"?per_page=100&page={page}&sort=updated&direction=desc"
    )

    if not batch:
        break

    repositories.extend(
        repository for repository in batch
        if not repository.get("fork", False)
    )

    if len(batch) < 100:
        break

    page += 1

# ---------------------------------------------------------------------
# Public GitHub events
# ---------------------------------------------------------------------

events = []
page = 1

while page <= 3:
    batch = get_json(
        f"{API}/users/{USERNAME}/events/public?per_page=100&page={page}"
    )

    if not batch:
        break

    events.extend(batch)

    if len(batch) < 100:
        break

    page += 1

events.sort(
    key=lambda event: event.get("created_at", ""),
    reverse=True,
)

now = datetime.now(timezone.utc)
year_start = datetime(now.year, 1, 1, tzinfo=timezone.utc)
last_30 = now - timedelta(days=30)
last_7 = now - timedelta(days=7)

def event_datetime(event):
    return datetime.fromisoformat(
        event["created_at"].replace("Z", "+00:00")
    )

year_events = [
    event for event in events
    if event.get("created_at") and event_datetime(event) >= year_start
]

month_events = [
    event for event in events
    if event.get("created_at") and event_datetime(event) >= last_30
]

week_events = [
    event for event in events
    if event.get("created_at") and event_datetime(event) >= last_7
]

# ---------------------------------------------------------------------
# Contribution calendar
# ---------------------------------------------------------------------

contribution_query = """
query($login:String!, $from:DateTime!, $to:DateTime!) {
  user(login:$login) {
    contributionsCollection(from:$from, to:$to) {
      contributionCalendar {
        totalContributions
        weeks {
          contributionDays {
            date
            contributionCount
          }
        }
      }
    }
  }
}
"""

try:
    contribution_data = graphql(
        contribution_query,
        {
            "login": USERNAME,
            "from": year_start.isoformat(),
            "to": now.isoformat(),
        },
    )

    calendar = (
        contribution_data["user"]
        ["contributionsCollection"]
        ["contributionCalendar"]
    )

    contribution_days = [
        day
        for week in calendar["weeks"]
        for day in week["contributionDays"]
    ]

    contributions_year = calendar["totalContributions"]

    contributions_30 = sum(
        day["contributionCount"]
        for day in contribution_days
        if datetime.fromisoformat(day["date"]).replace(
            tzinfo=timezone.utc
        ) >= last_30
    )

    contributions_7 = sum(
        day["contributionCount"]
        for day in contribution_days
        if datetime.fromisoformat(day["date"]).replace(
            tzinfo=timezone.utc
        ) >= last_7
    )

except Exception:
    contributions_year = len(year_events)
    contributions_30 = len(month_events)
    contributions_7 = len(week_events)

# ---------------------------------------------------------------------
# Manual project status
# ---------------------------------------------------------------------

with open(STATUS_FILE, "r", encoding="utf-8") as file:
    project_status = json.load(file)

# ---------------------------------------------------------------------
# Project table
# ---------------------------------------------------------------------

project_rows = []

for repository in repositories[:20]:
    name = repository["name"]
    description = (
        repository.get("description")
        or "No repository description."
    ).replace("|", "\\|")

    if len(description) > 90:
        description = description[:87] + "..."

    status = project_status.get(name, "Unspecified")

    project_rows.append([
        f"[{name}]({repository['html_url']})",
        description,
        repository.get("language") or "Multiple",
        status,
        relative_time(repository.get("updated_at")),
    ])

if not project_rows:
    project_rows.append([
        "No public repositories",
        "—",
        "—",
        "—",
        "—",
    ])

# ---------------------------------------------------------------------
# Currently building
# ---------------------------------------------------------------------

recent_push_repositories = {}

for event in events:
    if event.get("type") != "PushEvent":
        continue

    repository = event.get("repo", {}).get("name", "")
    if not repository.startswith(f"{USERNAME}/"):
        continue

    if repository not in recent_push_repositories:
        recent_push_repositories[repository] = event

current_rows = []

for full_name, event in list(
    recent_push_repositories.items()
)[:8]:

    repo_name = full_name.split("/", 1)[-1]

    repository = next(
        (
            item
            for item in repositories
            if item["name"] == repo_name
        ),
        None,
    )

    if not repository:
        continue

    description = (
        repository.get("description")
        or "Recently active repository."
    ).replace("|", "\\|")

    if len(description) > 85:
        description = description[:82] + "..."

    current_rows.append([
        f"[{repo_name}]({repository['html_url']})",
        description,
        event.get("payload", {}).get("ref", "push"),
        relative_time(event.get("created_at")),
    ])

if not current_rows:
    current_rows.append([
        "No recent public push activity",
        "—",
        "—",
        "—",
    ])

# ---------------------------------------------------------------------
# Daily activity
# ---------------------------------------------------------------------

activity_rows = []

for event in events[:12]:
    repository = event.get("repo", {}).get("name", "")

    if not repository:
        continue

    activity_rows.append([
        relative_time(event.get("created_at")),
        event_description(event),
        f"[{repository}](https://github.com/{repository})",
    ])

if not activity_rows:
    activity_rows.append([
        "No recent public activity",
        "—",
        "—",
    ])

# ---------------------------------------------------------------------
# Statistics
# ---------------------------------------------------------------------

stars = sum(
    repository.get("stargazers_count", 0)
    for repository in repositories
)

push_count = sum(
    1 for event in month_events
    if event.get("type") == "PushEvent"
)

pull_request_count = sum(
    1 for event in month_events
    if event.get("type") == "PullRequestEvent"
)

issue_count = sum(
    1 for event in month_events
    if event.get("type") == "IssuesEvent"
)

last_activity = (
    relative_time(events[0]["created_at"])
    if events
    else "No public activity"
)

stats_rows = [
    ["Public repositories", user.get("public_repos", 0)],
    ["Contributions this year", contributions_year],
    ["Contributions last 30 days", contributions_30],
    ["Commits / pushes", push_count],
    ["Pull requests", pull_request_count],
    ["Issues", issue_count],
    ["Stars", stars],
    ["Followers", user.get("followers", 0)],
    ["Last activity", last_activity],
    ["Last refresh", now.strftime("%Y-%m-%d %H:%M UTC")],
]

# ---------------------------------------------------------------------
# Update README
# ---------------------------------------------------------------------

with open(README_FILE, "r", encoding="utf-8") as file:
    readme = file.read()

readme = replace_section(
    readme,
    "PROJECTS",
    table(
        ["Project", "Description", "Tech", "Status", "Updated"],
        project_rows,
    ),
)

readme = replace_section(
    readme,
    "CURRENTLY_BUILDING",
    table(
        ["Repository", "Description", "Activity", "Last Updated"],
        current_rows,
    ),
)

readme = replace_section(
    readme,
    "DAILY_ACTIVITY",
    table(
        ["Time", "Activity", "Repository"],
        activity_rows,
    ),
)

readme = replace_section(
    readme,
    "GITHUB_STATS",
    table(
        ["Metric", "Value"],
        stats_rows,
    ),
)

with open(README_FILE, "w", encoding="utf-8") as file:
    file.write(readme)

print("GitHub profile README synchronized successfully.")
'''

(root / "README.md").write_text(readme, encoding="utf-8")
(root / "scripts" / "project_status.json").write_text(status, encoding="utf-8")
(root / ".github" / "workflows" / "sync-profile.yml").write_text(workflow, encoding="utf-8")
(root / "scripts" / "update_readme.py").write_text(script, encoding="utf-8")

zip_path = Path("/mnt/data/muhammad-arsalan-github-profile.zip")
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for file in [
        root / "README.md",
        root / "scripts" / "project_status.json",
        root / "scripts" / "update_readme.py",
        root / ".github" / "workflows" / "sync-profile.yml",
    ]:
        z.write(file, file.relative_to(root).as_posix())

print(zip_path)
