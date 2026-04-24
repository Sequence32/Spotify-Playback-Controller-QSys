# Spotify Playback Controller for Q-SYS

A professional Q-SYS plugin that provides full Spotify playback control through the Spotify Web API. Features OAuth PKCE authentication (no client secret needed), transport controls, now-playing metadata with inline album art display, a visual progress meter, playlist browsing, device selection, and a Spotify-branded dark UI theme.

![Q-SYS](https://img.shields.io/badge/Q--SYS-Plugin-blue)
![Spotify](https://img.shields.io/badge/Spotify-Web%20API-1DB954)
![Version](https://img.shields.io/badge/Version-1.4.3-orange)
![License](https://img.shields.io/badge/License-MIT-green)

> **🚧 Active Development** — This plugin is under active development as of April 2026. Features, controls, and authentication flows may change between versions. Feedback and contributions are welcome!

---

## Features

- **OAuth PKCE Authentication** — Secure login without exposing client secrets
- **Mobile-Friendly Login** — Browse to the Core's IP from any device; includes a code-paste fallback for remote auth
- **Inline Album Art** — Album artwork rendered directly in the plugin using Q-SYS Media controls
- **Visual Progress Bar** — Horizontal meter that tracks song playback in real time
- **Spotify-Themed UI** — Dark mode with Spotify green (#1DB954) accent colors on all controls
- **Transport Controls** — Play, Pause, Next, Previous, Shuffle, Repeat
- **Now Playing** — Track name, artist, album, progress text, and duration
- **Playlist Browsing** — View and play your top 10 playlists with cover art
- **Device Selection** — Browse and switch between up to 5 Spotify Connect devices
- **Seek** — Seek to any position in the current track
- **Auto Token Refresh** — Tokens refresh automatically before expiry
- **Full URL Paste** — Paste an entire redirect URL when manually entering auth codes; the plugin auto-extracts the code

---

## Prerequisites

- **Q-SYS Designer** 9.x or later
- **Spotify Premium** account (required for Web API playback control)
- A registered app in the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)

---

## Setup Guide

### Step 1 — Create a Spotify Developer App

1. Go to the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Log in with your Spotify account
3. Click **Create App**
4. Fill in:
   - **App Name**: anything (e.g., `Q-SYS Controller`)
   - **App Description**: anything
   - **Redirect URI**: `http://<core-ip>:9091/callback` (e.g., `http://192.168.1.6:9091/callback`)
   - **APIs Used**: check **Web API**
5. Click **Save**
6. Copy the **Client ID** from the app overview page

> **Important:** The redirect URI must match the Core's IP address. For emulation/development, use `http://127.0.0.1:9091/callback` or `http://localhost:9091/callback`. For a deployed Core on the LAN, add `http://<core-ip>:9091/callback` as a redirect URI in the Spotify Dashboard. You can add multiple redirect URIs.

### Step 2 — Install the Plugin in Q-SYS Designer

1. Copy `Spotify_Playback_Controller.qplug` to your Q-SYS plugins directory
2. Open your design in Q-SYS Designer
3. Find **Spotify Playback Controller** in the plugin list and drag it into your design

### Step 3 — Configure Plugin Properties

In the plugin's **Properties** panel, set:

| Property | Value | Notes |
|----------|-------|-------|
| **Client ID** | `your-spotify-client-id` | From the Spotify Developer Dashboard |
| **Core IP Address** | *(leave blank or set IP)* | Leave blank to auto-detect, or set the Core's LAN IP when deployed |
| **Redirect Port** | `9091` | Must match the port in your Spotify redirect URI |
| **Poll Interval (s)** | `3` | How often to poll for now-playing updates (1-10 seconds) |

### Step 4 — Authorize

#### From the same machine (Emulator)

1. Save/Emulate your design so the plugin is running
2. Press the **Authorize** button on the plugin block
3. The **Auth URL** field will show an address like `http://192.168.1.6:9091`
4. Open that address in any web browser on the same machine
5. Click the green **"Step 1: Log in with Spotify"** button
6. Log in to Spotify and approve the permissions
7. You'll see a "Spotify Authorized!" confirmation page — done!

#### From a phone or another device (deployed Core)

1. Press the **Authorize** button on the plugin
2. On your phone or tablet, open a browser and navigate to the **Auth URL** shown in the plugin (e.g., `http://192.168.1.6:9091`)
3. Tap **"Step 1: Log in with Spotify"** — this opens Spotify auth in a new tab
4. Log in and approve permissions
5. If the redirect is caught automatically, you're done!
6. If you see an error page after authorizing:
   - **Copy the entire URL** from the browser address bar
   - Go back to the landing page tab
   - **Paste the full URL** into the input field under "Step 2"
   - Tap **Submit Code**
7. You can also paste the full URL directly into the **Manual Code** field in the Q-SYS plugin

> **Tip:** You only need to authorize once. The plugin automatically refreshes the token before it expires. Re-authorization is only needed after a Core reboot or explicit logout.

---

## Controls Reference

### Authentication
| Control | Type | Description |
|---------|------|-------------|
| `Authorize` | Trigger | Start the OAuth login flow |
| `Logout` | Trigger | Clear tokens and stop polling |
| `Auth_Status` | LED Indicator | Green = authenticated, Red = logged out, Yellow = waiting |
| `Auth_URL` | Text (output) | Login page URL — open in any browser on the network |
| `Auth_Code_Input` | Text (input) | Paste a full redirect URL or raw auth code here as fallback |
| `User_Name` | Text (output) | Logged-in Spotify display name |

### Transport
| Control | Type | Description |
|---------|------|-------------|
| `Play` | Trigger | Resume playback |
| `Pause` | Trigger | Pause playback |
| `Next` | Trigger | Skip to next track |
| `Previous` | Trigger | Go to previous track |
| `Shuffle` | Toggle | Enable/disable shuffle mode |
| `Repeat` | Trigger | Cycle repeat: off → context → track → off |
| `Repeat_State` | Text (output) | Current repeat mode |
| `Is_Playing` | Indicator | True when playback is active |

### Now Playing
| Control | Type | Description |
|---------|------|-------------|
| `Track_Name` | Text (output) | Current track name |
| `Artist_Name` | Text (output) | Artist(s), comma-separated |
| `Album_Name` | Text (output) | Album name |
| `Album_Art_URL` | Text (output) | HTTPS URL for album artwork |
| `Album_Art` | Media (output) | Inline album art display rendered directly in the plugin |
| `Progress_Ms` | Meter 0-100% (output) | Visual progress bar tracking playback position |
| `Progress_Text` | Text (output) | Formatted as `2:34 / 4:12` |
| `Duration_Ms` | Text (output) | Track duration in milliseconds |

### Device & Seek
| Control | Type | Description |
|---------|------|-------------|
| `Active_Device` | Text (output) | Name of the active Spotify Connect device |
| `Device_ID` | Text (output) | Device ID (for debugging) |
| `Seek_Position` | Knob 0-100% (input) | Seek to a percentage of the track |

### Playlists (10 slots)
| Control | Type | Description |
|---------|------|-------------|
| `Refresh_Playlists` | Trigger | Fetch/refresh the user's playlists |
| `Playlist_Name 1-10` | Text (output) | Playlist names |
| `Playlist_Art 1-10` | Text (output) | Playlist cover art URLs |
| `Playlist_Play 1-10` | Trigger | Start playing that playlist |

### Device Selection (5 slots)
| Control | Type | Description |
|---------|------|-------------|
| `Refresh_Devices` | Trigger | Fetch available Spotify Connect devices |
| `Device_Name 1-5` | Text (output) | Device names (* marks active) |
| `Device_Type 1-5` | Text (output) | Device type (Computer, Smartphone, Speaker, etc.) |
| `Device_Select 1-5` | Trigger | Transfer playback to this device |

---

## UCI Integration

### Album Art on Touch Panels

The plugin renders album art directly in the plugin block using a **Media** style control. For UCI integration:

1. Add a **Media Display** element to your UCI layout
2. Set its source to the `Album_Art_URL` control
3. The plugin outputs HTTPS image URLs that Q-SYS can render natively

### Playlist Cover Art

Each `Playlist_Art 1-10` control contains the cover image URL for the corresponding playlist. Wire these to individual Media Display elements to show playlist thumbnails alongside the playlist names.

---

## Plugin UI Layout

The plugin features a Spotify-themed dark interface with the following sections:

| Section | Description |
|---------|-------------|
| **Authentication** | Authorize/Logout buttons, auth status LED, user name, auth URL, manual code input |
| **Now Playing** | 140x140 album art, track/artist/album metadata, progress text, progress meter bar |
| **Transport** | Play, Pause, Previous, Next buttons with Spotify green styling; Shuffle/Repeat toggles |
| **Active Device** | Currently active Spotify Connect device name |
| **Playlists** | 10-slot playlist browser with Play buttons |
| **Devices** | 5-slot device browser with Select buttons for playback transfer |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **"redirect_uri: Not matching"** | Ensure the redirect URI in Spotify Dashboard matches: `http://<core-ip>:9091/callback` |
| **Auth works on PC but not phone** | Add `http://<core-ip>:9091/callback` to the Spotify Dashboard redirect URIs. Use the landing page's code-paste form as fallback. |
| **"Invalid authorization code"** | Auth codes are single-use. Press Authorize again for a fresh code |
| **Transport buttons not working** | Ensure you have an active Spotify session on at least one device |
| **No album art in plugin** | The `Album_Art` control uses Media style — verify the plugin is receiving valid image URLs |
| **Plugin not loading / silent failure** | Check for syntax errors; ensure no unsupported Unicode characters in the .qplug file |
| **Plugin crashes on Authorize** | Check that the Client ID is set in Properties and the Core IP Address is correct |
| **Token expires after 1 hour** | Handled automatically — the plugin refreshes 60 seconds before expiry |

---

## Architecture

```
+-----------------------------------------------------+
|  Q-SYS Core / Designer Emulation                    |
|                                                     |
|  +---------------------------------------------+   |
|  |  Spotify Playback Controller Plugin          |   |
|  |                                              |   |
|  |  TcpSocketServer (:9091)                     |   |
|  |    |-- Serves login landing page             |   |
|  |    |-- Catches OAuth redirect callback       |   |
|  |    +-- Accepts manual code submission        |   |
|  |                                              |   |
|  |  HttpClient                                  |   |
|  |    |-- Token exchange & refresh              |   |
|  |    |-- Playback control (PUT/POST)           |   |
|  |    +-- Metadata polling (GET)                |   |
|  +---------------------------------------------+   |
|                                                     |
|  Browser (phone, tablet, or PC)                     |
|    +-- Opens login page at http://<core-ip>:9091    |
|    +-- Spotify OAuth -> redirect back to Core       |
|    +-- Fallback: paste URL into code form           |
+-----------------------------------------------------+
         |                          |
         v                          v
  Spotify Auth Server        Spotify Web API
  (accounts.spotify.com)     (api.spotify.com)
```

---

## Security Notes

- **No client secret stored** — Uses PKCE (Proof Key for Code Exchange), the recommended flow for public/embedded clients
- **Tokens are memory-only** — Access and refresh tokens are not persisted to disk; re-auth is needed after a Core reboot
- **Scopes are minimal** — Only requests permissions needed for playback control and playlist reading

---

## Changelog

### v1.4.3
- Accept full redirect URL paste in both Manual Code field and web form (auto-extracts auth code)
- Simplified landing page instructions for non-technical users

### v1.4.2
- Enhanced auth landing page with step-by-step instructions and code-paste form
- Added `/submit_code` route for manual auth from mobile devices
- Removed QR code feature (caused plugin loading issues)

### v1.3.2
- Fixed Lua parse error from Unicode escape sequences in button legends
- Replaced Unicode symbols with ASCII text for Q-SYS compatibility

### v1.3.0
- Added inline album art display using Q-SYS Media controls
- Added visual progress meter bar tracking song playback
- Applied Spotify-themed dark UI with green accent colors to all buttons and controls
- Added device selection (5 slots) with playback transfer

### v1.2.0
- Added Album Art control with Media style rendering

### v1.1.0
- Initial release with OAuth PKCE, transport controls, playlists, and now-playing metadata

---

## License

MIT License — See [LICENSE](LICENSE) for details.

---

## Author

**Dustin Bennett** — [TEL Systems](https://telsystems.com)
