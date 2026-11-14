# Weather Logger (n8n)

A small n8n workflow that fetches the current weather for a city from the [OpenWeatherMap](https://openweathermap.org/api) API and stores the result in a MySQL table.

## What it does

1. **Manual trigger:** you start it with *Execute workflow* (swap in a Schedule Trigger to run it automatically).
2. **HTTP Request:** calls the OpenWeatherMap "Current weather" endpoint for Karachi, in metric units.
3. **Edit Fields:** keeps only the city name, temperature and weather description.
4. **If:** continues only when the returned city name contains `Karachi`.
5. **MySQL (Update):** updates the matching row in the `weather_log` table with the new description and temperature.

```
Manual Trigger → HTTP Request → Edit Fields → If (city = Karachi) → MySQL Update
```

## Requirements

- An [n8n](https://n8n.io) instance (cloud or self-hosted)
- A free OpenWeatherMap API key
- A MySQL database the n8n instance can reach

## Database setup

The workflow **updates** an existing row. It does not insert one, so create the table and add a starting row first:

```sql
CREATE TABLE weather_log (
  city        VARCHAR(100) PRIMARY KEY,
  description VARCHAR(255),
  temperature DECIMAL(5,2)
);

INSERT INTO weather_log (city, description, temperature)
VALUES ('Karachi', NULL, NULL);
```

If the `city` value doesn't match the name OpenWeatherMap returns (for Karachi it returns `Karachi`), no rows will be updated.

## Setup

1. Import the workflow JSON in n8n (**Workflows → Import from file**).
2. In the **HTTP Request** node, replace `YOUR_OPENWEATHERMAP_API_KEY` in the URL with your key. For anything shared or deployed, store the key in an n8n credential instead of the URL.
3. In the **Update rows in a table** node, select your MySQL credential and the `weather_log` table.
4. Click **Execute workflow**. Then check the table:

```sql
SELECT * FROM weather_log;
```

## Customising

- **Different city:** change `q=Karachi` in the URL and the `Karachi` value in the **If** node. Add a matching row to the table.
- **Run automatically:** replace the Manual Trigger with a Schedule Trigger and activate the workflow.
- **Fahrenheit:** change `units=metric` to `units=imperial`.

## Notes

- The **Edit Fields** node lists the weather description field twice. This is harmless but can be cleaned up.
- The **If** node has no "false" branch, so the workflow simply stops if the city doesn't match.
- A new OpenWeatherMap key can take a little while to activate. Until then the API returns a 401 error.

## Security

Never commit API keys or database credentials. If a key has appeared in a public place, regenerate it in your OpenWeatherMap account.
