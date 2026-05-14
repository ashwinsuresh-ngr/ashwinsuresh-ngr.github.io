Title: Playlist Curator MCP Server
Date: 2026-04-22
Category: MCP
Tags: mcp, python, ytmusicapi, kiro, genai
Slug: building-playlist-curator-mcp-server


Building My Playlist Curator MCP Server with YouTube Music & Claude AI


When I first thought about music discovery, I was tired of switching tabs, searching manually, and letting algorithms decide what I wanted to hear. I wanted something simpler — just describe my mood and get music instantly. That's when I started building the Playlist Curator MCP Server, a project that connects Claude AI, YouTube Music, and Kiro IDE into one seamless experience.
This blog is a reflection of that journey — what MCP is, how I built the server, and why it completely changed the way I think about AI tools.


What is an MCP Server?

MCP stands for Model Context Protocol — an open standard that lets AI assistants like Claude connect to external tools and services. Instead of Claude just generating text, MCP gives it hands to reach out and interact with the real world.
In simple terms, MCP lets you build tools where AI doesn't just "talk" — it searches, creates, and takes actions on your behalf.
Why I Built This

Traditional music discovery is passive. You open Spotify or YouTube Music, scroll through suggestions, and hope the algorithm understands you. But I wanted something different:
A tool that understands natural language mood descriptions
Music search that gets context, not just keywords
Automatic playlist creation from a single sentence
A personal music assistant that lives inside my IDE
MCP made all of this possible.
How I Built It

Instead of relying on a paid API, I combined free tools into a powerful stack:

IDE: Kiro (AWS) for spec-driven development
MCP SDK: mcp (Python) to expose tools to Claude
Music API: ytmusicapi for YouTube Music access
Mood Engine: Claude AI to convert moods into search keywords
Language: Python 3.11
Auth: Browser cookie-based authentication
This combination kept the entire project free, fast, and completely personal.
The Magic: Turning Mood into Music
Here's where Claude truly shines.
Instead of a fixed mood-to-keyword dictionary, I used Claude itself as the recommendation engine. When you type a mood description, Claude converts it into smart YouTube Music search keywords.
So instead of searching for exact genres, you can say:
"Music that feels like a rainy Sunday in a café"
And Claude understands the context, not just the words — returning tracks that actually match the vibe.
The 5 MCP Tools I Built
The server exposes five tools that Claude can call:

search_by_mood

Takes a natural language mood description and returns matching tracks with title, artist, thumbnail, and video ID.
get_mood_playlist
Fetches pre-built YouTube Music mood and genre playlists directly.
get_recommendations
Given a seed track, returns similar songs using ytmusicapi's watch playlist.
create_playlist
Creates a real playlist in your YouTube Music account with a given name and tracks.
add_tracks
Adds new tracks to an existing playlist in your account.

Using It Inside Kiro
One of my favorite moments was testing this inside Kiro for the first time. You just chat naturally:
"Find me chill late night music"
"Save these as a playlist called Night Mode"
"Give me something similar to this track"
And the MCP server handles everything behind the scenes — searching, creating, saving — while Claude does the natural language understanding on top.

Challenges I Faced

It wasn't all smooth sailing. Some real challenges included:
Spotify dropped free API access — now requires Premium and limits test users to 5, which pushed me toward YouTube Music
Cookie format bug — Chrome exports cookies using -b flag but ytmusicapi expects -H 'cookie: ...' which caused a confusing error
Claude API vs subscription — found out that a Claude.ai Pro subscription doesn't include API access, they are completely separate products
Gzip-encoded request body — the copied cURL had binary compressed data that looked alarming but turned out to be harmless

Lessons Learned

If you're planning something similar, here's my advice:
Start with the spec files before writing any code
Pick free APIs whenever possible — ytmusicapi proved better than Spotify for this use case
Use Claude for the intelligent parts, not just as a chatbot
Keep authentication simple — browser cookies are powerful and free
Kiro's spec-driven workflow saves enormous time on boilerplate

The Future of MCP Tools

MCP servers are more than developer toys — they are the future of how AI integrates with real workflows.
With tools like ytmusicapi and the MCP protocol, we can build assistants that:
Understand personal context deeply
Take real actions on your behalf
Work inside the tools you already use every day
This is just the beginning.

Final Thoughts

Building the Playlist Curator MCP Server was more than a technical project — it was a shift in how I think about AI tools.
It made me realize that AI doesn't have to live in a separate tab or app. It can live right inside your IDE, understand your mood, and build something real for you in seconds.
If you're thinking about building your own MCP server, my advice is simple:
Start with one tool, keep it free, and let the idea grow naturally.
Thanks for reading! If you enjoyed this, feel free to explore the project on GitHub or try building your own MCP server — it's easier than you think.