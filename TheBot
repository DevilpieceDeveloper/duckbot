import discord
from discord import app_commands
import random
import requests
import json
from discord.ext import commands

# Read token from file
with open("duckbottoken.txt", "r") as file:
    TOKEN = file.read().strip()

GUILD_ID = 1335347194957398207  # Replace with your server ID
MODS = ["xr_ionic", "yorealducki", "yozeyastar"]  # Moderator usernames
XP_FILE = "xp_data.json"

WELCOME_CHANNEL_ID = 1335708011339190355  # Channel ID for welcome messages
GOODBYE_CHANNEL_ID = 1340422585695076463  # Channel ID for goodbye messages

intents = discord.Intents.default()
intents.messages = True
intents.reactions = True
intents.guilds = True
intents.message_content = True
client = commands.Bot(command_prefix="/", intents=intents)

# Load XP data
try:
    with open(XP_FILE, "r") as f:
        xp_data = json.load(f)
except FileNotFoundError:
    xp_data = {}

# Save XP data
def save_xp():
    with open(XP_FILE, "w") as f:
        json.dump(xp_data, f, indent=4)

# Calculate required XP for the next level
def required_xp(level):
    return 5 * (level ** 2) + 50 * level + 100

# Give XP to a user
def add_xp(user_id, username, amount):
    if username in MODS:
        xp_data[str(user_id)] = {"xp": float("inf"), "level": float("inf")}
        save_xp()
        return None
    
    if str(user_id) not in xp_data:
        xp_data[str(user_id)] = {"xp": 0, "level": 1}
    
    xp_data[str(user_id)]["xp"] += amount
    user_xp = xp_data[str(user_id)]["xp"]
    user_level = xp_data[str(user_id)]["level"]
    
    while user_xp >= required_xp(user_level):
        user_xp -= required_xp(user_level)
        user_level += 1
        xp_data[str(user_id)]["level"] = user_level
        save_xp()
        return user_level
    
    xp_data[str(user_id)]["xp"] = user_xp
    save_xp()
    return None

@client.event
async def on_ready():
    print(f'Logged in as {client.user}')
    try:
        synced = await client.tree.sync(guild=discord.Object(id=GUILD_ID))
        print(f'Synced {len(synced)} commands')
    except Exception as e:
        print(f'Error syncing commands: {e}')

@client.event
async def on_member_join(member):
    channel = client.get_channel(WELCOME_CHANNEL_ID)
    if channel:
        embed = discord.Embed(
            title=f"Welcome, {member.name}!",
            description="We're glad to have you here!",
            color=discord.Color.green()
        )
        embed.set_thumbnail(url=member.avatar.url)
        embed.set_footer(text="quack", icon_url=member.avatar.url)
        await channel.send(embed=embed)

@client.event
async def on_member_remove(member):
    channel = client.get_channel(GOODBYE_CHANNEL_ID)
    if channel:
        embed = discord.Embed(
            title=f"Goodbye, {member.name}!",
            description="Sorry to see you go. Hope to see you again soon!",
            color=discord.Color.red()
        )
        embed.set_thumbnail(url=member.avatar.url)
        await channel.send(embed=embed)

# /welcome_test command for testing welcome system
@client.tree.command(name="welcome_test", description="Test the welcome message with your avatar.", guild=discord.Object(id=GUILD_ID))
async def welcome_test(interaction: discord.Interaction):
    channel = client.get_channel(WELCOME_CHANNEL_ID)
    if channel:
        embed = discord.Embed(
            title=f"Welcome, {interaction.user.name}!",
            description="Testing the welcome message!",
            color=discord.Color.green()
        )
        embed.set_thumbnail(url=interaction.user.avatar.url)
        embed.set_footer(text="quack", icon_url=interaction.user.avatar.url)
        await interaction.response.send_message(embed=embed)

# /level command
@client.tree.command(name="level", description="Check your level.", guild=discord.Object(id=GUILD_ID))
async def level(interaction: discord.Interaction):
    user_id = str(interaction.user.id)
    if interaction.user.name in MODS:
        await interaction.response.send_message(f"{interaction.user.mention}, you have **infinite** level! 🦆🚀")
    elif user_id in xp_data:
        user_level = xp_data[user_id]["level"]
        await interaction.response.send_message(f"{interaction.user.mention}, you are at **Level {user_level}**! 🦆")
    else:
        await interaction.response.send_message(f"{interaction.user.mention}, you haven't earned any XP yet! Send messages or react to gain XP.")

# /roles command to display roles in embed
@client.tree.command(name="roles", description="Shows the roles in the server.", guild=discord.Object(id=GUILD_ID))
async def roles(interaction: discord.Interaction):
    roles = [role.name for role in interaction.guild.roles]
    embed = discord.Embed(
        title="Server Roles",
        description="Here are the roles in this server:",
        color=discord.Color.blue()
    )
    embed.add_field(name="Roles", value="\n".join(roles), inline=False)
    await interaction.response.send_message(embed=embed)

# /dadjoke command to send a random dad joke
@client.tree.command(name="dadjoke", description="Tells a random dad joke.", guild=discord.Object(id=GUILD_ID))
async def dadjoke(interaction: discord.Interaction):
    jokes = [
        "Why don’t skeletons fight each other? They don’t have the guts.",
        "I used to play piano by ear, but now I use my hands.",
        "What do you call fake spaghetti? An impasta."
    ]
    joke = random.choice(jokes)
    await interaction.response.send_message(joke)

# /say command to send a message as the bot
@client.tree.command(name="say", description="Have the bot say something.", guild=discord.Object(id=GUILD_ID))
@app_commands.describe(message="The message you want the bot to say")
async def say(interaction: discord.Interaction, message: str):
    await interaction.response.send_message(message)

# /purge command to delete a number of messages
@client.tree.command(name="purge", description="Delete a number of messages.", guild=discord.Object(id=GUILD_ID))
@app_commands.describe(amount="Number of messages to delete")
async def purge(interaction: discord.Interaction, amount: int):
    if interaction.user.permissions_in(interaction.channel).manage_messages:
        deleted = await interaction.channel.purge(limit=amount)
        await interaction.response.send_message(f"Deleted {len(deleted)} messages.", ephemeral=True)
    else:
        await interaction.response.send_message("You don't have permission to delete messages.", ephemeral=True)

# /ping command to check bot's response time
@client.tree.command(name="ping", description="Check the bot's ping.", guild=discord.Object(id=GUILD_ID))
async def ping(interaction: discord.Interaction):
    latency = round(client.latency * 1000)
    await interaction.response.send_message(f"Pong! Latency is {latency}ms.", ephemeral=True)

# /promote command to send clan promo message
@client.tree.command(name="promote", description="Sends the promotion message for the Ducky Clan.", guild=discord.Object(id=GUILD_ID))
async def promote(interaction: discord.Interaction):
    promo_message = """╭  🦆 𝘿𝙐𝘾𝙆𝙔 𝘾𝙇𝘼𝙉 🦆
🦆・Rare clan tag name [DUCKY]
🐥・LVL 50 BP, CLAN LVL 10, SHOP LVL 5 and more!
🦆・ 50 slots left!
╰  🦆   DM @Zeya, @Draco  or @quacker   for more details!"""
    await interaction.response.send_message(promo_message)

# /serverinfo command
@client.tree.command(name="serverinfo", description="Get information about the server.", guild=discord.Object(id=GUILD_ID))
async def serverinfo(interaction: discord.Interaction):
    guild = interaction.guild
    embed = discord.Embed(
        title=f"Server Information: {guild.name}",
        color=discord.Color.blue()
    )
    embed.add_field(name="Server ID", value=guild.id)
    embed.add_field(name="Total Members", value=guild.member_count)
    embed.add_field(name="Owner", value=f"{guild.owner} ({guild.owner_id})")
    embed.set_thumbnail(url=guild.icon.url if guild.icon else None)
    await interaction.response.send_message(embed=embed)

client.run(TOKEN)
