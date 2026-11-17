# Discord Bot Display Name Styles API Documentation

> **Note:** `display_name_styles` is an internal/undocumented Discord API field for bot accounts (documented via community API references like Discord Userdoccers). Behavior and availability on `users/@me` vs. `guilds/{guild_id}/members/@me` are subject to Discord API changes.

---

## 1. Endpoints

### Global Bot Profile Style
Applies the display name style globally across all servers where the bot does not have a per-server override.
* **Method:** `PATCH`
* **URL:** `https://discord.com`

### Per-Server (Guild) Bot Profile Style
Applies the display name style only inside a specific server.
* **Method:** `PATCH`
* **URL:** `https://discord.com{guild_id}/members/@me`

### Required Headers
```http
Authorization: Bot YOUR_BOT_TOKEN
Content-Type: application/json
```

---

## 2. Request Payload Schema

Send a JSON body containing the `display_name_styles` object.

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `display_name_styles` | `object` \| `null` | Yes | Style configuration object. Pass `null` to reset to default. |
| `font_id` | `integer` | Yes | ID of the font (`1` – `12`). |
| `effect_id` | `integer` | Yes | ID of the visual effect (`1` – `6`). |
| `colors` | `array<integer>` | Yes | Array of 24-bit RGB decimal integers (converted from hex). |

### Example JSON Payload (Global Style)
```json
{
  "display_name_styles": {
    "font_id": 8,
    "effect_id": 2,
    "colors": [16734003, 3399613]
  }
}
```

### Reset / Remove Style Payload
```json
{
  "display_name_styles": null
}
```

---

## 3. Reference Tables

### Font IDs (`font_id`)

| ID | Font Name | Discord UI Label |
| :--- | :--- | :--- |
| `1` | Bangers | Tempo |
| `2` | BioRhyme | Sakura |
| `3` | Cherry Bomb One | Jellybean |
| `4` | Chicle | Modern |
| `5` | Compagnon | Medieval |
| `6` | MuseoModerno | Mainframe |
| `7` | Neo Castel | Vampyre |
| `8` | Pixelify Sans | 8Bit |
| `9` | Ribes | Monkey Bars |
| `10` | Sinistre | Headbang |
| `11` | Default (`gg sans`) | Default |
| `12` | Zilla Slab | Journal |

### Effect IDs (`effect_id`)

| ID | Effect Name | Required `colors` Length | Notes |
| :--- | :--- | :--- | :--- |
| `1` | Solid | `1` color | Single solid text color |
| `2` | Gradient | `2` colors | Two-color gradient (`[startColor, endColor]`) |
| `3` | Neon | `1` color | Glowing neon outline effect |
| `4` | Toon | `1` color | Comic/cartoon shadow outline |
| `5` | Pop | `1` color | 3D extruded pop effect |
| `6` | Glow | `1` color | Soft ambient glow |

### Color Conversion Formula
Discord requires colors as **base-10 (decimal) integers**, not hex strings:
* Hex `#FF5733` → `0xFF5733` → `16734003`
* Hex `#33DFBD` → `0x33DFBD` → `3399613`

---

## 4. Code Examples

### cURL (Global Style)
```bash
curl -X PATCH "https://discord.com" \
  -H "Authorization: Bot YOUR_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "display_name_styles": {
      "font_id": 8,
      "effect_id": 2,
      "colors": [16734003, 3399613]
    }
  }'
```

### Node.js / JavaScript (Global & Guild Support)
```javascript
const BOT_TOKEN = "YOUR_BOT_TOKEN";

/**
 * Updates the Discord bot's Display Name Style.
 * @param {object} options
 * @param {number} options.fontId - Font ID (1-12)
 * @param {number} options.effectId - Effect ID (1-6)
 * @param {string[]} options.hexColors - Array of 1 or 2 hex color strings (e.g. ["#FF5733", "#33DFBD"])
 * @param {string|null} [options.guildId=null] - Optional Guild ID. Leave null for Global Bot Style.
 */
async function updateBotDisplayNameStyle({ fontId, effectId, hexColors, guildId = null }) {
  const endpoint = guildId
    ? `https://discord.com${guildId}/members/@me`
    : `https://discord.com`;

  const decimalColors = hexColors.map((hex) => parseInt(hex.replace("#", ""), 16));

  const response = await fetch(endpoint, {
    method: "PATCH",
    headers: {
      "Authorization": `Bot ${BOT_TOKEN}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      display_name_styles: {
        font_id: fontId,
        effect_id: effectId,
        colors: decimalColors,
      },
    }),
  });

  const data = await response.json();

  if (!response.ok) {
    throw new Error(`Discord API Error (${response.status}): ${JSON.stringify(data)}`);
  }

  return data;
}

// Example: Apply Global Bot Style (8Bit Font + Gradient)
updateBotDisplayNameStyle({
  fontId: 8,
  effectId: 2,
  hexColors: ["#FF5733", "#33DFBD"],
  guildId: null, // Global
})
  .then((res) => console.log("Updated Global Style:", res.display_name_styles))
  .catch(console.error);
```

### Python (`aiohttp`)
```python
import aiohttp
import asyncio

BOT_TOKEN = "YOUR_BOT_TOKEN"

async def update_bot_display_name_style(
    font_id: int,
    effect_id: int,
    hex_colors: list[str],
    guild_id: str | None = None
):
    url = (
        f"https://discord.com{guild_id}/members/@me"
        if guild_id
        else "https://discord.com"
    )

    decimal_colors = [int(color.lstrip("#"), 16) for color in hex_colors]

    payload = {
        "display_name_styles": {
            "font_id": font_id,
            "effect_id": effect_id,
            "colors": decimal_colors,
        }
    }

    headers = {
        "Authorization": f"Bot {BOT_TOKEN}",
        "Content-Type": "application/json",
    }

    async with aiohttp.ClientSession() as session:
        async with session.patch(url, json=payload, headers=headers) as resp:
            data = await resp.json()
            if resp.status >= 400:
                raise RuntimeError(f"Discord API Error ({resp.status}): {data}")
            return data

# Example: Apply Global Bot Style
async def main():
    result = await update_bot_display_name_style(
        font_id=8,
        effect_id=2,
        hex_colors=["#FF5733", "#33DFBD"],
        guild_id=None  # Global
    )
    print("Updated Global Style:", result.get("display_name_styles"))

asyncio.run(main())
```

---

## 5. Common Errors & Troubleshooting

* **`400 Bad Request` (`INVALID_FORM_BODY`):**
  * Verify `colors` has **2 integers** when `effect_id` is `2` (Gradient), and **1 integer** for all other effects (`1`, `3`, `4`, `5`, `6`).
  * Ensure `font_id` is between `1` and `12` and `effect_id` is between `1` and `6`.
* **`401 Unauthorized` / `403 Forbidden`:**
  * Ensure your `Authorization` header starts with `Bot ` followed by a valid bot token.
  * If Discord restricts `display_name_styles` on `users/@me` for bot accounts, test using the guild endpoint (`guilds/{guild_id}/members/@me`) instead.
* **`429 Too Many Requests`:**
  * Profile update endpoints have strict rate limits. Avoid calling this endpoint repeatedly in loops or on every bot startup.
