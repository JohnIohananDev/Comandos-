import discord
import json
import asyncio
from discord.ext import commands

bot = commands.Bot(command_prefix='!', intents=discord.Intents.all())

@bot.command()
@commands.has_permissions(administrator=True)
async def ban(ctx, member: discord.Member, *, motivo="Sem motivo"):
    await member.ban(reason=motivo)
    await ctx.send(f"o {member} foi **banido**. Motivo: {motivo}")

@bot.command()
@commands.has_permissions(administrator=True)
async def kick(ctx, member: discord.Member, *, motivo="Sem motivo"):
    await member.kick(reason=motivo)
    await ctx.send(f"o {member} foi **expulso**. Motivo: {motivo}")

@bot.command()
@commands.has_permissions(administrator=True)
async def mute(ctx, member: discord.Member, tempo: int = 10, *, motivo="Sem motivo"):
    mute_role = discord.utils.get(ctx.guild.roles, name="Muted")
    if not mute_role:
        mute_role = await ctx.guild.create_role(name="Muted")
        for channel in ctx.guild.channels:
            await channel.set_permissions(mute_role, send_messages=False, speak=False)
    await member.add_roles(mute_role, reason=motivo)
    await ctx.send(f"o {member} foi **mutado** por {tempo}min. Motivo: {motivo}")
    await asyncio.sleep(tempo * 60)
    await member.remove_roles(mute_role)

@bot.command()
@commands.has_permissions(administrator=True)
async def clear(ctx, quantidade: int = 10):
    await ctx.channel.purge(limit=quantidade + 1)
    await ctx.send(f"uma {quantidade} mensagens foi **apagadas**.", delete_after=3)

@bot.command()
@commands.has_permissions(administrator=True)
async def warn(ctx, member: discord.Member, *, motivo="Sem motivo"):
    try:
        with open('warns.json', 'r') as f:
            warns = json.load(f)
    except:
        warns = {}
    if str(member.id) not in warns:
        warns[str(member.id)] = []
    warns[str(member.id)].append(motivo)
    with open('warns.json', 'w') as f:
        json.dump(warns, f)
    await ctx.send(f"0 {member} levou um **warn**. Motivo: {motivo}")
