<template>
  <div id="app">

  <H1 style="color:blue;">REACH MORRA</H1>

    <H3 style="color:orange;"> Choose your role:</H3>
    <button class="buttoncolor" @click="agnes()" >Agnes</button> VS 
    <button class="buttoncolor" @click="benny()" >Benny</button>
    <BR/>
  
  <HR/>

  <div v-if="role==0">
  <h3>Agnes</h3>

    <button class="buttoncolor2" @click="createContract()" >Click to Deploy Contract</button><BR/>

    contract: (Copy below to Buyer and Transport)<BR/> 
    <h4>{{ ctcInfoStr }}</h4><BR /><BR />
  </div>

  <div v-else-if="role==1">
  <h3>Benny</h3>

  <input v-model="ctcStr" placeholder="paste contract here"> <button @click="attachContract()">Attach Contract</button>
  
    <BR/>  
    <div v-if="wager>0">
      Do you want to accept wager of {{wager}} ? 
      <button @click="yesnoWager(true)">YES</button>
      <button @click="yesnoWager(false)" >NO</button>
    </div>
    <BR/>     
  </div>

  <HR/>

    <div v-if="displayHandsState">
      <BR/>
      <h4>Last result are : </h4>
      <BR/>
      Agnes hand: {{agnesHands}}
      Agnes guess: {{agnesGuess}}
      <BR/>
      Benny hand: {{bennyHands}}
      Benny guess: {{bennyGuess}}
      <BR/>
  </div>

  <div v-if="getHandState">
      Play your hand : <BR/>
      <button @click="readHand(0)">ZERO</button>
      <button @click="readHand(1)">ONE</button>
      <button @click="readHand(2)">TWO</button> 
      <button @click="readHand(3)">THREE</button> 
      <button @click="readHand(4)">FOUR</button> 
      <button @click="readHand(5)">FIVE</button> 
      <BR/>
  </div>

  <div v-if="getGuessState">
      Shout your total : <BR/>
      <button @click="readGuess(0)"> 0 </button>
      <button @click="readGuess(1)"> 1 </button>
      <button @click="readGuess(2)"> 2 </button> 
      <button @click="readGuess(3)"> 3 </button> 
      <button @click="readGuess(4)"> 4 </button> 
      <button @click="readGuess(5)"> 5 </button> 
      <button @click="readGuess(6)"> 6 </button> 
      <button @click="readGuess(7)"> 7 </button> 
      <button @click="readGuess(8)"> 8 </button> 
      <button @click="readGuess(9)"> 9 </button> 
      <button @click="readGuess(10)"> 10 </button> 
      <BR/>
    </div>   

  <div v-if="displayResultState">
      <H3>{{resultString}}</H3>
  </div>
  
  <BR/>

  <HR/>
  addr: {{ addr}} <BR/>
  bal: {{ bal }} <BR/>
  balAtomic: {{ balAtomic }}<BR/>

  <button class="buttoncolor3" @click="updateBalance()">updateBalance</button>

  </div>
</template>

<script>
import * as backend from '../build/index.main.mjs';
import { loadStdlib } from '@reach-sh/stdlib';


const stdlib = loadStdlib("ALGO");
stdlib.setProviderByName("TestNet")

console.log(`The consensus network is ${stdlib.connector}.`);


const toSU = (au) => stdlib.formatCurrency(au, 4);


let commonInteract = { };
let agnesInteract = { };
let bennyInteract = { };

const OUTCOME = [ "NULL","Agnes Wins", "Benny Wins" ];

const secret = process.env.VUE_APP_SECRET1
const secret2 = process.env.VUE_APP_SECRET2

export default {
  name: 'App',
  components: {
  },
  data: () => {
    return {
      role: 0,
      acc: undefined,
      addr: undefined,
      balAtomic: undefined,
      bal: undefined,
      faucetLoading: false,
      ctc: undefined,
      ctcInfoStr: undefined,
      ctcStr: undefined,

      contractCreated: false,
      displayResultState: false,
      displayHandsState: false,
      getGuessState: false,
      getHandState: false,

      wager: undefined,
      hand: undefined,
      guess: undefined,
      agnesHands:undefined,
      agnesGuess:undefined,
      bennyHands:undefined,
      bennyGuess:undefined,
      resultString: undefined,
      acceptWager: undefined,
    };
  },
   methods: {

      commonFunctions() {

        commonInteract = {
            ...stdlib.hasRandom,
            reportResult: async (result) => { this.reportResult(result); },
            reportHands: (agnes, agnesGuess, benny, bennyGuess) => { this.reportHands(agnes, agnesGuess, benny, bennyGuess)},
            informTimeout: () => { this.informTimeout()},
            getHand: async () => {
                  console.log("*** getHand called from backend");
                  this.getHandState = true
                  await this.waitUntil(() => this.hand !== undefined );
                  console.log("You played ", this.hand + " finger(s)");
                  const hand = stdlib.parseCurrency(this.hand);
                  this.hand = undefined;
                  this.getHandState = false
                  return hand;
                },
            getGuess: async () => {
                  console.log("*** getGuess called from backend");
                  this.getGuessState = true

                  await this.waitUntil(() => this.guess !== undefined );
                  console.log("You guess total of ", this.guess);
                  const guess = stdlib.parseCurrency(this.guess);
                  this.guess = undefined;
                  this.getGuessState = false
                  return guess;
                },
          }
      },

      async reportResult(result) {
     
        console.log('*** reportResult ', result);
        this.resultString = OUTCOME[result];
        console.log('this.result ', this.resultString);
        this.displayResultState = true;
        await this.updateBalance();
      },

      reportHands(agnes, agnesGuess, benny, bennyGuess) {
        console.log('*** The hands are ' + agnes, agnesGuess, benny, bennyGuess );
        this.agnesHands = toSU(agnes);
        this.agnesGuess = toSU(agnesGuess);
        this.bennyHands = toSU(benny);
        this.bennyGuess = toSU(bennyGuess);

        this.displayHandsState = true;
      },

      readHand(hand) {
        console.log("readHand: ", hand)
        this.hand = hand;
      },

      readGuess(guess) {
        console.log("readguess: ", guess)
        this.guess = guess;
      },

    async createContract() {
        console.log("Creating contract...")
        this.ctc = await this.acc.contract(backend);
          this.ctc.p.Agnes(agnesInteract);

   
          const info = await this.ctc.getInfo();
          this.ctcInfoStr = JSON.stringify(info);
          console.log("this.ctcInfoStr: ", this.ctcInfoStr);

          this.contractCreated = true;
          await this.updateBalance();
    },

    async agnes() {
      this.commonFunctions();
      agnesInteract = {
            ...commonInteract,
            wager: stdlib.parseCurrency(1),
            deadline: stdlib.parseCurrency(10),
        }

      console.log("Agnes: ", agnesInteract);
      try {
          this.role = 0;
 
          this.acc = await stdlib.newAccountFromMnemonic(secret);

          this.addr = stdlib.formatAddress(this.acc.getAddress());

          this.balAtomic = await stdlib.balanceOf(this.acc);
          this.bal = String(stdlib.formatCurrency(this.balAtomic, 4));
            
      } catch (err) {
        console.log(err);
      }
    },
    

    async yesnoWager(res) {
        console.log("yesnoWager: ", res)
        this.acceptWager = res;
    },

   async attachContract() {
            this.ctc = await this.acc.contract(backend, JSON.parse(this.ctcStr));     
            console.log("Contract attached: ",this.ctcStr)
            await this.ctc.p.Benny(bennyInteract);
            await this.updateBalance();
    },

    async benny() {
      this.commonFunctions();
      bennyInteract = {
        ...commonInteract,
          acceptWager: async (wager) => {
          console.log("*** acceptWager", wager);

          this.wager =  toSU(wager),
          this.waitUntil( ()=> this.acceptWager == true)
          if ( this.acceptWager == false) {
              process.exit(0);
          }
        }
      }
      console.log("Robin: ", bennyInteract);

      try {
          this.role = 1;
  
          this.acc = await stdlib.newAccountFromMnemonic(secret2);

          this.addr = stdlib.formatAddress(this.acc.getAddress());

          this.balAtomic = await stdlib.balanceOf(this.acc);
          this.bal = String(stdlib.formatCurrency(this.balAtomic, 4));

      } catch (err) {
        console.log(err);
      }
    },  
    
    waitUntil (condition) {
    return new Promise((resolve) => {
        let interval = setInterval(() => {
            if (!condition()) {
                return
            }

            clearInterval(interval)
            resolve()
        }, 100)
      })
    },

    async updateBalance() {
      try {
        this.balAtomic = await stdlib.balanceOf(this.acc);
        this.bal = String(stdlib.formatCurrency(this.balAtomic, 4));
      } catch (err) {
        console.log(err);
      }
    },
  },
  computed: {
  },
  mounted() {
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#wizard {
  width: 1024px;
}
.content {
  width: 1024px;
}

.buttoncolor{
  background-color: blanchedalmond;
}

.buttoncolor2{
  background-color:lightsalmon;
}

.buttoncolor3{
  background-color:royalblue;
}


.navigation-buttons {
  display: flex;
  justify-content: space-between;
}
</style>
