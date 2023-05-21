import random
import requests
from bs4 import BeautifulSoup
import discord
from discord.ext import commands, tasks
import os
import sys
import json
from random import randint
from Cogs.SQLite_Schema import PlayerData, ServerSettings
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from Cogs.UserDataCog import AddNewUser, AddNewServer
import asyncio

import openai
openai.organization = "#################################"
openai.api_key = "################################################"

engine = create_engine("sqlite:///UserData.db", echo=True, future=True)
Session = sessionmaker(bind=engine)
session = Session()
FilePath = os.path.dirname(os.path.abspath(__file__))
intents = discord.Intents.all()
bot = discord.Client(intents=intents)


def GetSCP():
    while True:
        X = randint(2, 7999)
        if X < 100:
            Y = "00" + str(X)
        else:
            Y = str(X)
        r = requests.get(f'https://scp-wiki.wikidot.com/scp-{Y}')
        soup = BeautifulSoup(r.text, 'html.parser')
        try:
            rep = soup.find("img", style="width:300px;")
            link = rep.get("src")
            return link, Y
        except AttributeError:
            pass


class SCP5094(commands.Cog):
    def __init__(self, bot):
        self.bot = bot
        self.SCP = None
        self.CBmessage = None
        global SCPList
        SCPList = {}
        self.MessageTillTrigger = random.randint(1, 10)
        self.MessageCount = 0
        self.key = 0
        super().__init__()
    class SCPInput(discord.ui.Modal, title="SCP Quiz"):
        Guess = discord.ui.TextInput(label="What is your guess?", placeholder="SCP-", style=discord.TextStyle.short,
                                     required=True, max_length=20)
        async def on_submit(self, interaction: discord.Interaction,):
            await interaction.response.defer()
            await SCP5094.QuizLogic(self,Guess=self.Guess,member=interaction.user)

    class GuessButton(discord.ui.View):
        def __init__(self):
            super().__init__(timeout=600)

        @discord.ui.button(label="Take a Guess", style=discord.ButtonStyle.green, disabled=False)
        async def GuessButton(self, interaction, button):
            await interaction.response.defer()
            await interaction.response.send_modal(SCP5094.SCPInput())

    async def QuizLogic(self, Guess, member):
        for key in SCPList:
            if str(Guess).__contains__(key):
                embed2 = discord.Embed(title=f"SCP-{key} was correct",
                                       description=f"Good job {member.name} ",
                                       color=0x1f87fa)
                embed2.set_thumbnail(url="https://i.imgur.com/DYlgzDN.png")

                User = session.query(PlayerData).filter(PlayerData.User_ID == member.id).first()
                try:
                    X = User.QuizPoints
                    X = X + 1
                    User.QuizPoints = X
                    session.commit()
                except AttributeError:
                    U = member.id
                    AddNewUser(U)
                    User = session.query(PlayerData).filter(PlayerData.User_ID == member.id).first()
                    X = User.QuizPoints
                    X = X + 1
                    User.QuizPoints = X
                    session.commit()
                message = SCPList.get(key)
                self.key=key
                await message.edit(embed=embed2,view=None, delete_after=600)

        SCPList.pop(self.key)

    @commands.command()
    async def Ask(self,ctx, *text):
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=text,
            temperature=0.85,
            max_tokens=256,
            top_p=1,
            frequency_penalty=0.66,
            presence_penalty=0.54,
            user=ctx.author.name
        )
        message = await ctx.send(response["choices"][0]["text"])


    @commands.is_owner()
    @commands.command()
    async def PicTest(self, guild):
        if len(SCPList) == 9:
            return
        Link, Y = GetSCP()
        print(Y)

        channel = discord.utils.get(guild.text_channels, name='quiz_room')
        #        RolesChannel = await ctx.guild.create_text_channel(self.ChannelName)
        embed = discord.Embed(title="Pop Quiz",
                              description="Can you identify the SCP related to this photo?",
                              color=0x1f87fa)

        embed.set_image(url=Link)
        message = await channel.send(embed=embed,view=self.GuessButton(),delete_after=600)
        SCPList[Y] = message

    @commands.Cog.listener()
    async def on_message_delete(self, message):
        if message.author == self.bot.user:
            for key in SCPList:
                if message == SCPList.get(key):
                    SCPList.pop(key)
                    break
            print(SCPList)

    @commands.Cog.listener()
    async def on_message(self, message):
        if message.author == self.bot.user:
            return
        self.MessageCount += 1
        if self.MessageCount == self.MessageTillTrigger:
            await self.PicTest(message.guild)
            self.MessageTillTrigger = random.randint(1, 10)
            self.MessageCount = 0


async def setup(bot):
    await bot.add_cog(SCP5094(bot))
