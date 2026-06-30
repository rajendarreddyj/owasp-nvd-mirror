# OWASP NVD Mirror

A GitHub Actions-powered tool that periodically downloads and mirrors the [NVD (National Vulnerability Database)](https://nvd.nist.gov/) used by [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/). The database is packaged as a compressed archive and published as a GitHub Release, providing a ready-to-use mirror for anyone who wants to skip the slow NVD download process.

## Why?

OWASP Dependency-Check needs to download the full NVD database before scanning dependencies. This process can be slow and is rate-limited by the NVD API. This project solves the problem by providing a pre-built, regularly updated database that you can download and use directly.

## How It Works

1. A GitHub Actions workflow runs **every Monday at 02:00 UTC** (also supports manual trigger).
2. It uses the `dependency-check-maven` plugin to download/update the full NVD database.
3. The database is cached across runs for incremental updates.
4. The result is compressed into `owasp-nvd-database.tar.gz` and published as a GitHub Release under the tag `owasp-nvd-data-latest`.

## Download

Download the latest database archive from the [Releases page](https://github.com/rajendarreddyj/owasp-nvd-mirror/releases/tag/owasp-nvd-data-latest).

## Usage

1. Download and extract the archive:

   ```bash
   wget https://github.com/rajendarreddyj/owasp-nvd-mirror/releases/download/owasp-nvd-data-latest/owasp-nvd-database.tar.gz
   tar -xzf owasp-nvd-database.tar.gz
   ```

2. Point your `dependency-check-maven` configuration to the extracted data directory:

   **Maven CLI:**
   ```bash
   mvn org.owasp:dependency-check-maven:check \
     -DdataDirectory=./dc-data \
     -Dscan=./your-project
   ```

   **pom.xml:**
   ```xml
   <plugin>
     <groupId>org.owasp</groupId>
     <artifactId>dependency-check-maven</artifactId>
     <version>12.2.2</version>
     <configuration>
       <dataDirectory>/path/to/dc-data</dataDirectory>
     </configuration>
   </plugin>
   ```

## Tech Stack

- **Java 21** (Temurin)
- **Maven** + OWASP Dependency-Check Maven Plugin 12.2.2
- **GitHub Actions** (scheduled CI pipeline)
- **GitHub Releases** (artifact distribution)

## Configuration

| Setting | Value | Description |
|---------|-------|-------------|
| Schedule | `0 2 * * 1` (Mon 02:00 UTC) | Weekly automatic update |
| NVD API Delay | 500ms | Rate limit compliance |
| Data Directory | `dc-data/` | NVD database storage location |
| Release Tag | `nvd-data-latest` | Always points to the latest build |
| Timeout | 720 minutes | Maximum workflow duration |

### Required Secrets

- `NVD_API_KEY` — Your [NVD API key](https://nvd.nist.gov/developers/request-an-api-key) for authenticated access with higher rate limits.
- `GITHUB_TOKEN` — Automatically provided by GitHub Actions.

## Forking

To set up your own mirror:

1. Fork this repository.
2. Add your `NVD_API_KEY` to the repository secrets (**Settings > Secrets and variables > Actions**).
3. Enable GitHub Actions for the forked repository.
4. The workflow will run on schedule, or you can trigger it manually from the Actions tab.
