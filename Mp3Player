# -*- coding: utf-8-*-
import random
import re
import os
from client import jasperpath

WORDS = ["MUSIC"]


def getRandomSong(filename=jasperpath.data('text', 'SONGS.txt')):
    songFile = open(filename, "r")
    songs = []
    start = ""
    end = ""
    for line in songFile.readlines():
        line = line.replace("\n", "")

        if start == "":
            start = line
            continue

        if end == "":
            end = line
            continue

        songs.append((start, end))
        start = ""
        end = ""

    songs.append((start, end))
    song = random.choice(songs)
    return song


def handle(text, mic, profile):
    """
        Arguments:
        text -- user-input, typically transcribed speech
        mic -- used to interact with the user (for both input and output)
        profile -- contains information related to the user (e.g., phone
                   number)
    """

    song = getRandomSong()

    mic.say(song[0])
    
    os.system("mpg123 %s" % song[1])

    
def isValid(text):
    """
        Arguments:
        text -- user-input, typically transcribed speech
    """
    return bool(re.search(r'\bmusic\b', text, re.IGNORECASE))
