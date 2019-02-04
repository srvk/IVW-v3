Hello! Welcome to the Interactions in Virtual Worlds (IVW) machine.

Firstly, you will need to setup a host only network on the vm. (Virtual box is recommended, because we use the default network it creates in the code.) For more information on how to do this visit http://speech-kitchen.org/virtualbox-advanced-setup/

Now start the vm by clicking the monkey. You could also run the start_backend.sh script by typing the following commands into a new terminal.

cd Desktop
sh start_backend.sh

This will open four windows: Parser Wrapper, OpenSimulator, Kaldi Online Decoder (gst), and gui-demo.py. Wait for the OpenSimulator window to say "INITIALIZATION COMPLETE FOR Master" before proceeding. IMPORTANT: To exit the opensim virtual world server type "shutdown". DON'T JUST KILL IT!! If you don't shut it down properly it will leave objects in the virtual world half-written-to, which will cause the Friend Bot to fail in strange ways. 

The next step is to log into the virtual world as World Master. Go back to the host computer, and open Singularity viewer. Go to grid manager then to grids. Click the create button. You should use the address 192.168.56.101:9000 (which is the host-only network you created). The password for "World Master" is "avatar" and the passwords for Friend Bot is "alive". Log in as World Master. You should be able to interact with the world. 

Now, in the vm, open Monodevelop and open the solution IVW. Click run, and you should see the Friend Bot appear in the peninsula in the middle of the world. He will follow you around, and will ask you if you would like to brew potions.

Now, to have the system pick up your speech, go to the gui-demo.py window in the vm and click speak. You should see your speech transcribed in the same window. 

A quick aside about what is going on. What gets transcribed to this window goes into a log file in ~/Desktop/gst-kaldi-nnet2-online/demo/ and is read into the Parser Wrapper. The Parser Wrapper parses the speech into a dictionary of all kinds of useful things like semantic role labeling and syntax trees. For this system, we only use the semantic role labeling (srl) and the list of words in the sentence. This dictionary is transferred via a socket server to the Friend Bot, where it is used.

Back to the demo. If you respond yes to the Friend Bot, he will ask you if you would like an explanation. Answering yes again will get you one, and answering no will take you straight to the potion making.

Another quick aside. The Dialog that the Friend Bot follows can be easily edited in the ~/Desktop/cmd_transcription/dialog_transitions_and_actions.json file. The Friend Bot has three states: Normal, Explain, and Brew. The objects denoted in this file describe the functions that actions that are possible in each state, along with the actions commands that change the state and an animation (i.e. function) that is called when the state changes.

Once you are in the Brew state, there are a few things you can do. You can go find an ingredient on the island and bring it back to the potion table, then say something like "could you please add ___ to the potion". The potion is essentially a stack. Adding an ingredient to the stack pushes it to the top, and when you stir the potion ("please stir the potion") if the top two ingredients can mix, they are both popped of and their result is pushed on the top. Here is an example of making a potion.

add gunpowder to the potion
add foutain water too
stir the potion
add charcoal to the potion
now add fountain water
stir the potion again
stir it one more time

The result of this should be fire.

If you want to read/edit the explanation of potion making or the logic directly, its all in the ~/Desktop/cmd_transcription/ file. 

How the potion bot interprets the parsed data it gets can also be edited in the ~/Desktop/cmd_transcription/parse_to_commands_list.json file. "V" is the verb that is detected. "A1" is the direct object of that verb. (A0 is the subject and A2 is the indirect object.) If the srl that is passed in matches to commands, the one with the higher priority will be executed. If you want to know what the parser will spit out for a given input, you can use it directly with python. Try entering the following commands into a new terminal.

mario@Mario:~$ python
Python 2.7.6 (default, Jun 22 2015, 17:58:13) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from practnlptools.tools import Annotator
>>> an = Annotator()
>>> s = "explain potion making again"
>>> an.getAnnotations(s)['srl']
[{'A1': 'potion making', 'AM-TMP': 'again', 'V': 'explain'}, {'A1': 'potion', 'V': 'making'}]

So to add the sentence "explain potion making again", you know that the "V" needs to be "explain". (You could add an "A1" also, but I don't think it's necessary.)

You can also execute functions directly from the chat box. For example try typing "addIngredient pond water". Or "jump". Any function in the Action Client class can be called this way. Not just the ones in the current state. If you type a function name that does not exist, the Application Output of Monodevelop should say "Object reference not set to an instance of an object."

If you want to add your own functions, just write them in the ActionClient, then add the appropriate entries to parse_to_commands_list.json and dialog_transitions_and_actions.json.

You can look at the Next Steps file for more ideas if you play through this demo.

I think that's everything. For more information you can check out speech-kitchen.org.
