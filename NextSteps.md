Here are a few things you could do after you've played through the potion making demo.

1) You could make the friend bot recognize other commands in the Action Client. To do this you should first figure out what the parser spits out from the input sentence you want to give it. Lets say you want to add the command "jump" or "please jump". First open a terminal and enter the following commands.

python
from practnlptools.tools import Annotator
an = Annotator()
s = "jump please"
an.getAnnotations(s)['srl']

The result should look like this.

[{'A1': 'please', 'V': 'jump'}]

The 'V' means verb and the 'A1' means direct object. To make it so the Friend Bot can understand this sentence, you need to make two edits. The first is to make an entry to the parse_to_commands_list.json file in the cmd_transcription folder. Open it up, and you should see two types of entries. There should be "simple" and "srl" entries. srl stands for semantic role labeling, which in our case evaluates to [{'A1': 'please', 'V': 'jump'}]. Your entry should look something like this.

{
	"kind":"srl",
	"V":["jump", "hop", "leap"],
	"command":"jump",
	"priority":0
},

The list ["jump", "hop", "leap"] are all the possible verbs. The "command" is the function in the Action Client that gets called (if you go in the Action Client, you will see that there is a function named jump). The "priority" is just a way of deciding which command should be called in to are possible fits. This won't be a problem for us, so we can just use 0. Let's say we wanted to make it so the Friend Bot would only jump if we asked politely. We could edit our entry to look like this.

{
	"kind":"srl",
	"V":["jump", "hop", "leap"],
	"A1":["please"],
	"command":"jump",
	"priority":0
},

The "A1" is a list of all the possible direct objects. Now the command will only be called if the Friend Bot sees "please" in the "A1" position. 

The next step is to edit the dialog_transitions_and_actions.json file (also in the cmd_transcription folder). Now you have to add the action "jump" to one of the dialog states, so the Friend Bot knows when it should and shouldn't accept the "jump" command. Let's say I want my Friend Bot to only jump when he is not busy making potions. I should then add "jump" to the "Normal" state. The correct entry would look like this.

{
	"kind":"Action",
	"state":"Normal",
	"cmd":"jump"
},

Entries of the kind "Action" add an action to one of the Friend Bot's dialog states. With this and the previous edit in place, The Friend Bot should do the jump command when the user speaks in the Normal state. Entries of the kind "Transition" denote when and how to change the state. Let's say I wanted to change the state to Brew when the Friend Bot jumps. I could add the following entry.

{
	"kind":"Transition",
	"state1":"Normal",
	"state2":"Brew",
	"cmd":"jump",
	"animation":"say Ok now i'm changing states"
},

The "cmd" is the action that triggers the state change, and the animation is an additional function that is also called when the state is changed. Here the animation is the function "say" with the arguments "Ok", "now", "i'm", "changing", and "states".  

There are lots of ready made functions in the Action Client that you can use to make you Friend Bot do different things.

2) You could combine commands and make new ones. To do this you can add functions to the Action Client, which is written in C#. For example, I could make a function "jumpThreeTimes" that might look like this.

public void jumpThreeTimes(string[] paras) {
       for(int i=0; i<3; i++) {
       	       jump(null);
	}
}

Or a jumpThreeTimesThenSitDown function that looks like this.

public void jumpThreeTimes(string[] paras) {
       for(int i=0; i<3; i++) {
       	       jump(null);
	}
	sitDown(null);
}

3) You could edit the potion logic. Say I wanted to make it so that you could make fire out of only charcoal and gun powder. I would open the following line to the potion_operations.json file.

"charcoal:gun powder":"fire",

It is important that you list the two input ingredients in alphabetical order. You can only mix two ingredients at a time, but if you want to make more complicated potions, you can make intermediary ingredients (like I did with  stewed ___). For example, if I wanted to make fire out of gun powder, charcoal, and cherry tree, I could add the following lines.

"charcoal:gun powder":"spark dust",
"cherry tree:spark dust":"fire",

Make whatever kind of potion making rules you want!
