# Elixir Ecosystem 2020 reponses data
This repo contains the raw and processed data from the responses of [The Elixir Ecosystem Survey 2020](https://elixirsurvey.typeform.com/report/yYmJv1/OcCCilUmDn8lBpgP).

# Why this repo?
We have all this data from the Elixir Ecosystem Survey 2020 (thanks https://github.com/bcardarella for that). But, the raw data is not easy to query it, explore and make visualizations.

So, this repo contains a processed version of that data, with a data model that make it easy to query it.

# What this repo contains?

This repo contains two files:

- **elixir-ecosystem-survey-2020-processed-data.dump**: this file contains a PostgreSQL dump with the processed data. You can load that to a PostgreSQL instance and start exploring it
- **elixir-ecosystem-survey-2020-raw-data.csv**: this file contains the original CSV export from the survey (you can also find the [original file here](https://drive.google.com/file/d/1iddghuuob9_e9CFm05VnHjlELiwgnQqz/view))

# Query examples

Here are two examples of queries you can run:

```sql
-- Is there any relationship between how the dev debug and the number of years using elixir?

SELECT
	r. "How long have you been using Elixir?",
	sum((r. "How do you debug?" -> 'IO.{puts, inspect}')::BIGINT) AS "Debug using IO.{puts, inspect}",
	sum((r. "How do you debug?" -> 'IEx.pry')::BIGINT) AS "Debug using IEx.pry",
	sum((r. "How do you debug?" -> 'break!')::BIGINT) AS "Debug using break!",
	sum((r. "How do you debug?" -> ':debugger')::BIGINT) AS "Debug using :debugger",
	count(r. "How do you debug?" -> 'Other') AS "Debug using Other"
FROM
	"responses" r
WHERE
	r. "How do you debug?" -> 'Other' IS NOT NULL
	AND r. "How do you debug?" -> 'Other' <> '0'
GROUP BY
	1
ORDER BY
	1
```

```sql
-- Is there any relationship between companies using Elixir and the number of years the person uses Elixir?
SELECT
	*
FROM
  crosstab (
    $$
      SELECT
        r. "How long have you been using Elixir?",
        r. "Does your company use Elixir?",
        COUNT(r. "Does your company use Elixir?")
      FROM
        "responses" r
      WHERE
        r. "How long have you been using Elixir?" <> ''
      GROUP BY
        1,
        2
      ORDER BY
        1,
        2 $$
    ,
    $$
      VALUES(FALSE), (TRUE)
    $$
  ) AS ct ("How long have you been using Elixir?" TEXT, "Company DOESN't use Elixir" BIGINT, "Company USES Elixir" BIGINT)
```
