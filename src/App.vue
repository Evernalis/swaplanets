  <script>

const serverUrl='wss://swaptrumpserver.sfatkin2009.workers.dev/websocket';

import { isOn } from '@vue/shared';
import planicon from './components/planicon.vue'
import statbox from './components/statbox.vue';
function weightedRandom(weights) {
    function summation(sum, num){ return sum + num} ;
    let checkvalue=Math.random()*weights.reduce(summation, 0);
    let runningtotal=0
    let i=0;
    for(let j=0; j<weights.length;j++) { // iterates through weights until selection is reached
        
        runningtotal -= 0 - weights[j];
        if(runningtotal >= checkvalue) {
            console.log(weights);
            console.log(weights[j]);
            console.log(runningtotal);
            console.log(checkvalue);
            return i
        }
        i++

    }
}
let deckSize=42;





 let planetlist;
        let biomelist = [];
        function getPlanets() {
                fetch("https://swapi.info/api/planets/")
                    .then(res => {return res.json()})
                    .then(data => {console.log(null);
                    planetlist = data.filter(filterlist);
                    for (let i=0; i<planetlist.length; i++) {
                        if (planetlist[i].population != 'unknown'){
                           planetlist[i].population /= 1000000;
                        }
                    }
                    function filterlist(value) {
                    let nullcount=0;
                    if (value.orbital_period=='unknown') {nullcount++};
                    if (value.rotation_period=='unknown') {nullcount++};
                    if (value.population=='unknown') {nullcount++};
                    if (value.diameter=='unknown') {nullcount++};
                    if (value.surface_water=='unknown') {nullcount++; value.surface_water=0};
                    return nullcount<=1 // only use planets with 1 or 0 unknown
                    }
                    console.log(planetlist)
                    
        for(let planet of planetlist){          // get list of unique biome types
            for(let biome of planet.terrain.split(', ')) {
                if(!biomelist.includes(biome)){
                    biomelist.push(biome) 
                }
            }
        }
        for(let planet of planetlist){
            if(planet.terrain.split(', ').length==1) {console.log(planet.terrain)}
        }
        console.log(biomelist);})
                }
        getPlanets()
        
        function shuffle(arr) {
            let shuffled=[];
            let safelist = arr;
            let listlength = safelist.length;
            for(let i=0; i<listlength; i++){
                shuffled.push(safelist.splice(Math.floor(Math.random()*safelist.length),1)[0])
            }
            return shuffled
        } 
        
        let shufflist=[]
        {for(let i=0; i<42; i++) {
            shufflist.push(i)
        }}
        let p2deck=shuffle(shufflist);
        
        console.log(deckSize);
        let p1deck;
        console.log(p1deck);
        console.log(p2deck);
        
       
        



        export default {
          provide() {
              return {
                biolist: biomelist // sends a global variable to planicon 
              }
          },
            data() {
                return {
                    result: '',
                    pName: 'No planet yet!',
                    transkey: {
            'name':'This shouldn\'t be here.',
            'rotation_period': 'Day length',
            'orbital_period':'Orbit length',
            'diameter':'Diameter',
            'climate':'Climate',   
            'gravity':'Relative planet gravity',
            'terrain':'Terrain types',
            'surface_water':'Percentage of water cover',
            'population':'Population in millions',
            'residents': 'Residents',
            'films':'Featured in',
            'created':'This shouldn\'t be here.',
            'edited':'This shouldn\'t be here.',
            'url':'This shouldn\'t be here.'
        },
        
        descript: 'default description',
        validcats: ['diameter', 'orbital_period','rotation_period','surface_water','population'],
        biometype: [],
                    planetid: 0,
                    choice: undefined,
                    responsename: null,
                    responseval: null,
                    outcome:'no',
                    inComparison: false,
                    active: '1',
                    drawpile: [],
                    p1len: 21,
                    p2len: 21,
                    isAI: true,
                    aiflag:'(AI)',
                    stupidity: 0.2,
                    scrolltarget: 0,
                    scrolliter:0,
                    evald:false,
                    blist: biomelist,
                    response: undefined,
                    rollEvent: false,
                    netlink: undefined,
                    isOnline: false,
                    decksLocked: false,
                    rivname: 'Player Two',
                    username1: 'Player One',
                    isSecondClient: false,
                    pdat1: undefined,
                    pdat2: undefined
                }
            },
            components: {
              planicon,
              statbox
            },
            methods: {
                connectserver() {     // Establishes a connection. This is called when the Online Mode button is pressed.
                    this.username1=null;
                    while (this.username1==null) {
                        this.username1 = window.prompt('Please enter your username to continue.')
                    }
                    this.netlink = new WebSocket(serverUrl)
                    this.netlink.addEventListener("open", () => {
                    console.log("CONNECTED");
                    this.isOnline=true;
                    this.isAI=false;
                    this.netlink.send(JSON.stringify({ // mail is discarded if unread, this is intentional
                        mailtype: 'initial',
                        uName: this.username1
                    }))});
                    this.netlink.addEventListener('message', (msg) => this.getMail(JSON.parse(msg.data)))
                    
                },
                getMail(mail) { // function called on any message from other client
                    console.log('Mail recieved with type ' + mail.mailtype)
                    console.log('Raw data:')
                    console.log(mail)
                    if(mail.mailtype=='initial') { // message recieved when a second user makes inital contact
                        this.isOnline=true;
                        this.decksLocked = true
                        p2deck.splice(deckSize, 42-deckSize);
                        p1deck=p2deck.splice(0,Math.floor(p2deck.length/2));
                        this.rivname=mail.uName;
                        this.netlink.send(JSON.stringify({
                            mailtype: 'handshake',
                            decks: [p1deck, p2deck],
                            uName: this.username1,
                            decklen:deckSize
                        })
                        )
                    } else if(mail.mailtype=='handshake') { // message when the first client acknowledges the existence of the second, syncs and initialises
                        this.isOnline=true;
                        document.getElementById('planInput').value=mail.decklen
                        deckSize=mail.decklen;
                        this.decksLocked=true;
                        p1deck=mail.decks[0];
                        p2deck=mail.decks[1];
                        console.log(p2deck);
                        this.rivname=mail.uName;
                        this.active='2';
                        this.isSecondClient=true;
                    } else if(mail.mailtype=='choice'){// message when the other client has chosen a category
                        this.choice=mail.choice;
                        this.comparetrumps(false, true);
                    } else if(mail.mailtype=='eval') {// message when the other client has advanced to the next round, mirrors for both clients
                        this.evaluation(true)
                    } else if(mail.mailtype=='start') {// same as above, but for game start
                        this.displayNextPlanet(false)
                    }
                
                },
                displayRandomPlanet() { //unused
                    let rawplanet;
                    let planetfetch=Math.floor(Math.random()*planetlist.length);
                    this.planetid=planetfetch;
                    data=planetlist[planetfetch]
                        this.result = data;
                    //    console.log(data);
                        this.pName = this.result.name                  
                    
                   // console.log(this.pName)
                },
                displayNextPlanet(bump=false) {// now only used at the start of the game
                    if(bump==true && this.isOnline) {// used to prevent perpetual dialogue
                        console.log('posting')
                        this.netlink.send(JSON.stringify({
                            mailtype:'start'
                        }))
                    }
                    console.log('loading.')
                    if (!this.isOnline) {
                    p2deck.splice(deckSize, 42-deckSize);
                    p1deck=p2deck.splice(0,Math.floor(p2deck.length/2))}
                     // chop deck in local, done in onMail in online

                     
                    this.result=planetlist[p1deck[0]];
                    console.log(this.result);
                    console.log(this.result.name);
                    this.pName=this.result.name;
                    this.p1len=p1deck.length;
                    this.p2len=deckSize-this.p1len-this.drawpile.length;
                        
                    if(this.isSecondClient) {
                        this.pdat2=this.result;
                    } else {
                        this.pdat1=this.result;
                    }
                    if(this.isSecondClient){
                    this.pdat1=planetlist[p2deck[0]]
                    } else {
                    this.pdat2=planetlist[p2deck[0]]
                    }
                    
                },
                logid() {
                    console.log(this.planetid)
                },
                updateCardCount(){ //menu edit reaction
                    if (document.getElementById('planInput').value < 2) {
                        document.getElementById('planInput').value = 2
                    } else if (document.getElementById('planInput').value > 42) {
                        document.getElementById('planInput').value=42
                    } else {
                        deckSize=document.getElementById('planInput').value
                    } 
                },
                updateStupidity(){ //menu edit reaction
                    document.getElementById('stupidityselector').value
                },
                toggleAI(){ //menu edit reaction
                    this.isAI=!this.isAI;
                    if(this.isAI) {
                        this.aiflag='(AI)'
                    } else {
                        this.aiflag=''
                    }
                },
                rollingchoice(){  //recursively cycles through options as part of the effect when robot is deciding on category
                    if (this.scrolliter==10+this.scrolltarget){
                        console.log(this.scrolltarget);
                        this.comparetrumps(true); //runs regular comparison code
                        return
                    }
                    this.scrolliter++;
                    this.choice=['rotation_period','orbital_period','diameter','surface_water','population'][this.scrolliter%5];
                    setTimeout(this.rollingchoice, 100) //delayed loop
                },
                loopback(val){this.rollingchoice(val)},
                comparetrumps(scrolled=false, fromMail=false) {  //run when 'Compare Button' is pressed
                    if (this.isOnline && !fromMail) {
                        this.netlink.send(JSON.stringify({
                            mailtype: 'choice',
                            choice: this.choice
                        }))
                    }
                    this.inComparison=true;
                    if(this.isAI&&this.active=='2'&&!scrolled&& !this.isOnline){  // if AI needs to choose redirect then teriminate, call again when chosen after rollingchoice
                        this.aiCompare();
                        return null;
                    } //
                    this.rollEvent=false;
                    let chosenval=this.result[this.choice];
                    if (chosenval=='unknown'){ // data marked as unknown is set to zero
                        chosenval=0
                    }
                    
                    this.response=planetlist[p2deck[0]]; // handles second client info in online
                    if(this.isSecondClient) {
                        this.pdat1=this.response;
                    } else {
                        this.pdat2=this.response;
                    }
                    this.responseval=planetlist[p2deck[0]][this.choice];
                    console.log(this.choice);
                    console.log(this.responseval)

                    this.picon2dat=planetlist[p2deck[0]].terrain;
                    if (this.responseval=='unknown') {
                        this.responseval=0
                    }
                    this.responsename=planetlist[p2deck[0]].name; 
                    let namelist=[this.username1,this.rivname]
                    if(chosenval - 1 == this.responseval - 1) {         //subtraction of 1 forces numeric type to avoid string comparison issues
                        this.outcome="It's a draw!"
                    } else if((chosenval - 1 < this.responseval - 1) != ((this.active=='1') == this.isSecondClient)){
                        this.outcome='!AX has been dethroned!'.replace('!AX', namelist[this.active-1])
                    } else {
                        this.outcome='!AX wins this round.'.replace('!AX', namelist[this.active-1])
                    }
                    this.evald=true;
                    console.log(planetlist[p2deck[0]])
                },
                
                aiCompare(){ // rollingchoice() is the function that sets this.choice then redirects, this initiates rollingchoice
                  this.rollEvent=true;
                  this.response=planetlist[p2deck[0]];
                    let winlist =[];
                    for(let i=0; i<5; i++){         //evaluates potential outcomes
                    let chosenval=this.result[this.validcats[i]];
                    if (chosenval=='unknown'){
                        chosenval=0
                    }
                    let airesponseval=planetlist[p2deck[0]][this.validcats[i]];
                    if (airesponseval=='unknown') {
                        airesponseval=0
                    }
                    if(chosenval - 1 == airesponseval - 1) {
                        winlist[i]='Draw'
                    } else if((chosenval - 1 < airesponseval - 1)){
                        winlist[i]='Loss'
                    } else {
                        winlist[i]='Win'
                    }}
                    
                    //Decision making bit
                    let catWeights=[0,0,0,0,0]
                    function checkfunc(value) {return value=='Win'}
                    let winModifier=0.4/winlist.filter(checkfunc).length
                    for (let i=0; i<5; i++){    //computes each category as a weight 
                        if (winlist[i]=='Draw'){
                            catWeights[i]=0.8
                        } else if (winlist[i]=='Win') {
                            catWeights[i]=0.4+winModifier
                        } else {
                            catWeights[i]=this.stupidity
                        }
                        console.log('weight at '+i+' is '+ catWeights);
                    }
                    this.scrolltarget=weightedRandom(catWeights);
                    this.scrolliter=0;
                    this.rollingchoice()
                },
                evaluation(fromMail=false){             //runs to start the next round of the game, when 'Continue' is pressed.
                if (this.isOnline && !fromMail) { //prevents perpetual dialogue
                        this.netlink.send(JSON.stringify({
                            mailtype: 'eval',
                        }))
                    }
                    this.evald=false;
                    if (this.outcome == "It's a draw!"){
                        this.drawpile.push(p1deck.shift());
                        this.drawpile.push(p2deck.shift()); //pools both cards 
                    } else{
                        
                        if (this.outcome[this.outcome.length - 1] == '!') { //dethrone event always ends with !
                            let temp=p1deck.splice(0);
                            p1deck=p2deck.splice(0); //swaps the decks, so p1deck is always the active player
                            p2deck=temp.splice(0);
                            console.log('flipping')
                            this.active={'1':'2','2':'1'}[this.active] //swaps active between 1 and 2
                        }
                        p1deck.concat(this.drawpile); //gives pooled cards to winner
                        this.drawpile=[];
                        p1deck.push(p2deck.shift()) // gives rival card to winner
                    }
                    p1deck.push(p1deck.shift()) //cycles card to front
                    this.inComparison=false;
                    this.result=planetlist[p1deck[0]];
                    this.pName=this.result.name;
                    this.choice=null;
                    this.p1len=p1deck.length
                    this.p2len=deckSize-this.p1len-this.drawpile.length ;
                    console.log(this.active)
                    if(this.p2len==0 || this.p1len==0) { //active player has all cards
                        alert('XXX wins!'.replace('XXX', {'1': this.username1, '2': this.rivname}[this.active]));
                        window,location.reload()
                    } 
                if(this.isSecondClient) { //handles icons for clients in online
                        this.pdat2=this.result;
                    } else {
                        this.pdat1=this.result;
                    }
                if(this.isSecondClient){
                    this.pdat1=planetlist[p2deck[0]]
                } else {
                    this.pdat2=planetlist[p2deck[0]]
                }
                },
                upate(val) { // event from statbox to forward variable
                  this.choice=val;
                }
            }
        };
</script>

<!--this.active=='2' == --> 






<template>
        <div id="startmenu" v-if=" pName =='No planet yet!'">
        <br>
        <button  @click="displayNextPlanet(true)" :disabled="isOnline&& !decksLocked" :class="{highlight:decksLocked}">Click to Start!</button> <span v-if="isOnline&& !decksLocked">Awaiting second player.</span> <span v-if="decksLocked">Player found.</span>
        <button @click="toggleAI" :class="{highlight: !isAI}" :disabled="isOnline">Enable local multiplayer mode</button>
        <button @click="connectserver" :disabled="isOnline">Connect to Online Mode</button>
        <span > &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AI Difficulty:  </span>
        <select id="stupidityselector" @change="updateStupidity" :disabled="!isAI || isOnline">
            <option value="0.69">Easy</option>
            <option value="0.43" selected>Moderate</option>
            <option value="0.21">Hard</option>
            <option value="0.06">Omniscient</option>
        </select><span> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Number of planets(&lt;=42): </span> <input :disabled="decksLocked" id="planInput" value=42 @change="updateCardCount" type="number">
        <br></div>
        
        
        
        <div v-if="pName!='No planet yet!'">
            <statbox id="awakebox" :class="{rivbox: active==2, activebox: active==1}" v-if="pName!='No planet yet!'" :selectable="!(active==2&&(isAI||isOnline))" :planetdata="pdat1" @upate="(val) => upate(val)" :locked="evald||inComparison" :overridehighlight="choice"/>
            
                <div class="controlbox">
              <div>
                <button @click="comparetrumps()" id="comparebtn" :class="{vanish: !(((isAI||isOnline)&&active=='2'&&!inComparison)||(!inComparison&&pName!='No planet yet!'&&choice!=null))}" :disabled="!((isAI&&active=='2'&&!inComparison)||(!inComparison&&pName!='No planet yet!'&&choice!=null))">Compare!</button>  
              
                
                  
                <button @click="evaluation()" v-if="evald">Continue</button>
            <br>
            
                <h2 v-if="evald">Rival Planet: {{responsename}}!</h2>
                <p v-if="evald">{{responsename}} has a {{transkey[choice]}} value of {{responseval}}!</p>
                <p v-if="evald">{{outcome}}         </p>
              </div>
              </div>
            
            <statbox :class="{rivbox: active==1, activebox: active==2}" :rolling="rollEvent||(isOnline&&active=='2'&&!(inComparison||evald))" :planetdata="pdat2" :mystery="(!evald && !inComparison)" :locked="evald" :overridehighlight="choice" :selectable="false" />
                <!-- rivbox and activebox switch positions instead of function when player changes-->
                <div style="height:40px;  " />  <div class="nameplate" :class="{rivplate:active==2}">{{ username1 }}</div><div class="nameplate":class="{rivplate:active=='1'}">{{ rivname }}</div> <br> <div  class="cardpile">{{p1len}}</div><div  class="cardpile p2pile">{{ p2len }}</div> 
            </div>
                <br><br> 
                



</template> 




<style>
    body {
        background-image:url('./assets/planetimage/bgimage.png');
        background-size:auto;
        background-color: #ccbbaa;
        padding: 0px;
        margin:0;
        /*overflow: hidden;*/
    }
    #topborder {
        background-color: rgb(184, 184, 184);
        font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
        font-size: 150%;
        height: 50px;
        left: 0px;
        padding-top: 5px;
        margin: 0px;
        width:100%;
        margin-top: 0px;
        margin-bottom: 50px;
        border-style: solid;
        color: rgb(255, 247, 0);
        border-bottom-left-radius: 30px;
        border-bottom-right-radius: 30px;
        text-align: center;
        justify-self: center;
        box-sizing: border-box;
        /*position:absolute;*/  
        
    
    }
    button {
        padding: 10px;
        border-radius: 20px;
        border-width: 2px;
        margin-right: 10px;
        transition-duration: 200ms;
        border-style: solid;
        

    }
    button:hover {
        border-color:#000000;
        border-style: double;
    }
    #planInput {
        width: 30px;
    }
    #pdisp {
        background-image: radial-gradient(circle, rgba(255, 255, 200, 0.5),  rgb(196, 196, 196));
        border-style: double;
        border-color: black;
        border-radius: 5px;
        border-width: 1px;
        max-width: 90%;
        padding-left:5%;
        margin-left:5%;
        margin-right:5%;
        margin-top:1.5%;
        margin-bottom: 0;
        transition-duration: 1500ms;
        height:40em;
        padding-top: 15px;  
        
       
    }
    .cardpile {
        background-image: url('./assets/planetimage/stackpic.png');
        background-repeat: no-repeat;
        height: 128px;
        width: 128px;
        font-size: 35px;
        font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
        color: #ffdd99;
        text-align: center;
        padding-top:30px;
        display: inline-block;
        margin-left:10%;
        margin-top: 1%;
       

    }
    .p2pile {
        float:right;
        background-image: url('./assets/planetimage/stackpicred.png');
        margin-right:15%;
        margin-left: 30%;
    }
    .controlbox{
      background-image:linear-gradient(#aaaaaa, #a3883e);
      width: 25%;
      margin: 2%;
      margin-left: 5%;
      height: 300px;
      display: inline-flex;
      float:inline;
      border-radius: 20px;
      border-color:#ffff00;
      border-style: solid;
      left: 0%;  
      padding: 5px;
      margin-top: 0;
      margin-bottom: 0;
      transition-duration: 0;
    }
    #comparebtn {
      border-color: azure;
      position:relative;    
      margin: 0;
      height:30px;
      float:none;
      margin-bottom: -0px;
      margin-top: 0px;
      padding-top: 8px;
      align-items: center;

    }
    #comparebtn.vanish  {
      width: 0px;
      padding: 0px;
    }
    .nameplate{
        background-color: hsl(198, 25%, 50%);
        width: 20%;
        margin-left:5%;
        height: 40px;
        border-style: solid;
        border-color:hsl(198, 25%, 30%);
        font-size: 30px;
        font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
        color: #dbd9d7;
        text-align: center;
        display:inline-block;
        
    }
    .rivplate{
        float:right;
        margin-left:0%;
        margin-right: 10%;
        margin-top:0px;
    }
    .selector{
        background-color:#FCFCFC;
        border-radius: 10px;
        border-width:1px;
        justify-content: center;
        width: 20px;
        padding: 0px;
        padding-left: 0px;
        padding-top:1px;
        text-overflow: clip;
        height: 20px;
        float: left;
        margin-right: 0px;
        transition-duration: 500ms;
        margin-top:0px;
        display: inline;
        left: -1px;
    }
    .selector:hover{
        border-style: double;
        border-color:#ffffff;
        background-image: radial-gradient(#ffffff, #ffff00);
    }
    .statbox {
        transition-duration: 2s;
        border-style: solid;
        border-radius: 8px;
        border-color: rgb(248, 244, 0);
        background-image: linear-gradient(rgb(170, 170, 170), #848c71);
        padding-left:-10px;
        padding-top: 10px;
        padding-bottom: 10px;
        padding-right: 10px;
        transition-duration: 100ms;
        margin:0;
        width:30%;
        height:350px;
        line-height: auto;
        vertical-align: text-top;
        float:left;
        box-sizing: border-box;
    }
    .listitem{
        background-image: linear-gradient(to right, #444444, rgb(143, 143, 0) 40%,rgb(143, 143, 0),#444444);
        margin-top: 5px;
        margin-bottom: 5px;
        margin-left:-25px;
        margin-right: 0px;
        object-position: 15px 0px ;
        padding-left: 0px;
        padding-right:-30px; 
        line-height: normal;
        transition-duration: 500ms;
        display: block;
        border-radius:20px;
        font-family:'Courier New', Courier, monospace;
        border-style: solid;
        border-color: #697282;
        color: #ffffff;
        overflow: hidden;
        height:19px;
        max-width: 98%;
        
    }
    .highlight {
        background-image: linear-gradient(to right, #777755, rgb(255, 255, 0) 40%,rgb(255, 255, 0),#777755);
        color: #222221;
        


    }
    .vanish {
        background-color: rgba(0, 0, 0, 0);
        background-image: none;
        color: rgba(0, 0, 0, 0);   
        border-style: none; 
        border-width: 0px;
        font-size: 0px;
        border-color:#00000000;
        border-radius: 1px;
        width: 0px;
        margin-right: 55px;
        margin-bottom: 0px; 
        margin-left:10px;
        display:inline;
        margin-top: 9px;
        padding-bottom:-10px;
        height:0px;
    }
    .pullback {
        margin-right:-10px;
        width: auto;
        color: #00000000
    }
    #startmenu {
        width: 90%;
        height: 80px;
        display: inline-block;
        transition-duration: 500ms;
        background-color: #777755;
        padding:5px;
        justify-content: center;
        border-style: ridge;
        border-bottom-right-radius: 30px;
        border-bottom-left-radius: 30px; 
    }
    
    #graphicbox{
        margin-bottom: 144px;
    }
    #comparebtn {
        display: inline;
    }
    .rivbox {
        margin-left:0% ;
        float: right;
        margin-right: 5%;
        transition-duration: 0;
    }
    .activebox {
      float: left;
    }
    #awakebox {
      border-style: double;
      border-width: 8px;
      border-radius: 16px;


    }

</style>
