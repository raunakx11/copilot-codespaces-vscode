from telethon import TelegramClient, events, functions
from googletrans import Translator  # For translating text
from datetime import datetime
import time

# Replace with your API credentials
api_id = "22814806"
api_hash = "05c7b09110e0554684f4c6e230907367"

# Initialize the Telegram client
client = TelegramClient("selfbot", api_id, api_hash)
translator = Translator()  # Translator instance

# Tracks muted chats
muted_chats = set()

@client.on(events.NewMessage(outgoing=True))
async def outgoing_handler(event):
    global muted_chats
    command = event.raw_text.strip()

    # Command: .cmd
    if command == ".cmd":
        commands_list = (
            "📜 **Command List:**\n\n"
            "👋 `.me` - Displays your introduction.\n"
            "🔇 `.mute` - Mutes the chat.\n"
            "🔊 `.unmute` - Unmutes the chat.\n"
            "🚫 `.block` - Blocks a user (reply to the user's message).\n"
            "✅ `.unblock` - Unblocks a user (reply to the user's message).\n"
            "🗑 `.del` - Deletes the message.\n"
            "🧹 `.purge` - Deletes all messages in the chat.\n"
            "ℹ️ `.info` - Displays user info (reply to a user's message).\n"
            "🔗 `.gc` - Provides the group link.\n"
            "📷 `.insta` - Shows the Instagram handle.\n"
            "⏰ `.time` - Displays the current time.\n"
            "👨‍💻 `.devloper` - Shows the developer's username.\n"
            "📨 `.spam <count> <message>` - Sends a message multiple times.\n"
            "🌐 `.trans <text> (<language>)` - Translates text into the specified language.\n"
            "❓ `.cmd` - Displays this command list.\n"
        )
        await event.edit(commands_list)

    # Command: .me
    elif command == ".me":
        await event.edit("👋 Hey, greadIy here!")

    # Command: .mute
    elif command == ".mute":
        muted_chats.add(event.chat_id)
        await event.edit("🔇 This chat is muted. Messages will be deleted after mute.")

    # Command: .unmute
    elif command == ".unmute":
        if event.chat_id in muted_chats:
            muted_chats.remove(event.chat_id)
            await event.edit("🔊 This chat is now unmuted.")
        else:
            await event.edit("This chat is not muted.")

    # Command: .block
    elif command == ".block":
        if event.is_reply:
            user = await (await event.get_reply_message()).get_sender()
            await client(functions.contacts.BlockRequest(user.id))
            await event.edit(f"🚫 Blocked {user.first_name}.")
        else:
            await event.edit("Reply to a user to block them.")

    # Command: .unblock
    elif command == ".unblock":
        if event.is_reply:
            user = await (await event.get_reply_message()).get_sender()
            await client(functions.contacts.UnblockRequest(user.id))
            await event.edit(f"✅ Unblocked {user.first_name}.")
        else:
            await event.edit("Reply to a user to unblock them.")

    # Command: .del
    elif command == ".del":
        await event.delete()

    # Command: .purge
    elif command == ".purge":
        async for message in client.iter_messages(event.chat_id):
            await message.delete()
        await event.respond("🧹 Chat purged.")

    # Command: .info
    elif command == ".info":
        if event.is_reply:
            user = await (await event.get_reply_message()).get_sender()
            user_info = (
                f"ℹ️ **User Info**:\n\n"
                f"👤 **ID**: {user.id}\n"
                f"📛 **Username**: @{user.username}\n"
                f"📜 **Name**: {user.first_name}\n"
                f"📞 **Phone**: {user.phone}\n"
                f"🤖 **Bot**: {'Yes' if user.bot else 'No'}"
            )
            await event.edit(user_info)
        else:
            await event.edit("Reply to a user to get their info.")

    # Command: .gc
    elif command == ".gc":
        await event.edit("🔗 Join here: https://t.me/teamgread")

    # Command: .spam
    elif command.startswith(".spam"):
        try:
            parts = command.split(maxsplit=2)
            if len(parts) < 3:
                await event.edit("Usage: .spam <count> <message>")
                return

            count = int(parts[1])
            message = parts[2]

            for _ in range(count):
                await client.send_message(event.chat_id, message)
            await event.delete()
        except ValueError:
            await event.edit("❌ Invalid count. Usage: .spam <count> <message>")
        except Exception as e:
            await event.edit(f"⚠️ Error: {e}")

    # Command: .time
    elif command == ".time":
        now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        await event.edit(f"⏰ Current time: {now}")

    # Command: .insta
    elif command == ".insta":
        await event.edit("📷 @greadIy")

    # Command: .devloper
    elif command == ".devloper":
        await event.edit("👨‍💻 @greadIy")

    # Command: .trans
    elif command.startswith(".trans"):
        try:
            parts = command.split(maxsplit=2)
            if len(parts) < 3:
                await event.edit("Usage: .trans <text> (<language>)")
                return

            text, lang = parts[1], parts[2].strip("()")
            translated = translator.translate(text, dest=lang)
            await event.edit(f"🌐 Translated to {lang}: {translated.text}")
        except Exception as e:
            await event.edit(f"⚠️ Translation Error: {e}")

    # Handle muted chats
    elif event.chat_id in muted_chats:
        await event.delete()

@client.on(events.NewMessage(incoming=True))
async def incoming_handler(event):
    global muted_chats

    # Automatically delete messages from muted chats (DMs or Groups)
    if event.chat_id in muted_chats:
        await event.delete()

@client.on(events.ChatAction())
async def chat_action_handler(event):
    """
    Handles chat actions like someone joining or leaving in a group.
    """
    if event.user_added or event.user_joined:
        # Handle someone being added or joining
        await event.respond("👋 Welcome to the group! Use commands responsibly.")

# Start the client
print("Self-bot is running...")
client.start()
client.run_until_disconnected()

<!--
  <<< Author notes: Course header >>>
  Read <https://skills.github.com/quickstart> for more information about how to build courses using this template.
  Include a 1280×640 image, course name in sentence case, and a concise description in emphasis.
  In your repository settings: enable template repository, add your 1280×640 social image, auto delete head branches.
  Next to "About", add description & tags; disable releases, packages, & environments.
  Add your open source license, GitHub uses the MIT license.
-->

# Code with GitHub Copilot

_GitHub Copilot can help you code by offering autocomplete-style suggestions right in VS Code and Codespaces._

</header>

<!--
  <<< Author notes: Course start >>>
  Include start button, a note about Actions minutes,
  and tell the learner why they should take the course.
-->

## Welcome

GitHub Copilot can help you code by offering autocomplete-style suggestions. You can learn how GitHub Copilot works, and what to consider while using GitHub Copilot. GitHub Copilot analyzes the context in the file you are editing, as well as related files, and offers suggestions from within your text editor. GitHub Copilot is powered by OpenAI Codex, a new AI system created by OpenAI.

- **Who this is for**: Developers, DevOps Engineers, Software development managers, Testers.
- **What you'll learn**: How to install Copilot into a Codespace, accept suggestions from code, accept suggestions from comments.
- **What you'll build**: Javascript files that will have code generated by Copilot AI for code and comment suggestions.
- **Prerequisites**: 
  - [GitHub account](https://github.com/login) - With available Codespaces minutes.
  - [GitHub Copilot](https://github.com/github-copilot/signup) - For learning, the **Copilot Free** option with usage limits should be sufficient.
- **Timing**: This course can be completed in under an hour.

### How to start this course

<!-- For start course, run in JavaScript:
'https://github.com/new?' + new URLSearchParams({
  template_owner: 'skills',
  template_name: 'copilot-codespaces-vscode',
  owner: '@me',
  name: 'skills-copilot-codespaces-vscode',
  description: 'My clone repository',
  visibility: 'public',
}).toString()
-->

[![start-course](https://user-images.githubusercontent.com/1221423/235727646-4a590299-ffe5-480d-8cd5-8194ea184546.svg)](https://github.com/new?template_owner=skills&template_name=copilot-codespaces-vscode&owner=%40me&name=skills-copilot-codespaces-vscode&description=My+clone+repository&visibility=public)

1. Right-click **Start course** and open the link in a new tab.
2. In the new tab, most of the prompts will automatically fill in for you.
   - For owner, choose your personal account or an organization to host the repository.
   - We recommend creating a public repository, as private repositories will [use Actions minutes](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions).
   - Scroll down and click the **Create repository** button at the bottom of the form.
3. After your new repository is created, wait about 20 seconds, then refresh the page. Follow the step-by-step instructions in the new repository's README.

<footer>

<!--
  <<< Author notes: Footer >>>
  Add a link to get support, GitHub status page, code of conduct, license link.
-->

---

Get help: [Post in our discussion board](https://github.com/orgs/skills/discussions/categories/code-with-copilot) &bull; [Review the GitHub status page](https://www.githubstatus.com/)

&copy; 2023 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

</footer>
