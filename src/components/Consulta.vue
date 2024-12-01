<template>
  <main>
    <aside>
      <Sidebar menu="0" />
    </aside>
    <section class="bg-dark bg-gradient">
      <div class="header text-white d-flex align-items-center p-3 mb-1 rounded">
        <h1>Consultar Diplomas</h1>
      </div>
      <div class="content">
        <div class="form p-4 rounded">
          <div class="info-address d-flex gap-4">
            <div class="input-group mt-5">
              <span class="input-group-text" id="inputGroup-sizing-default">Endereço do aluno</span>
              <input type="text" class="form-control" aria-label="Sizing example input"
                aria-describedby="inputGroup-sizing-default" v-model="address_searched">
              <button type="button" @click="searchDegree()" class="btn btn-primary rounded">Consultar</button>
            </div>
          </div>

          <hr class="border border-secondary border-2">

          <div class="input-group info-student">
            <div class="card" style="width: 18rem;" v-if="address_founded != null">
              <div class="card-body">
                <h5 class="card-title">Address: {{ address_founded }}</h5>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<script setup>
import Web3 from 'web3';
import { ref, onMounted } from 'vue';
import Sidebar from '@/components/SideBar.vue';
import Swal from 'sweetalert2';

const default_value_wallet = 0x0000000000000000000000000000000000000000000000000000000000000000;
const web3 = new Web3('http://127.0.0.1:8545/');
const address = "0x5FbDB2315678afecb367f032d93F642f64180aa3";
const abi = [
  {
    "inputs": [],
    "stateMutability": "nonpayable",
    "type": "constructor"
  },
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "internalType": "address",
        "name": "_from",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "address",
        "name": "_to",
        "type": "address"
      },
      {
        "indexed": false,
        "internalType": "bytes32",
        "name": "_hash",
        "type": "bytes32"
      }
    ],
    "name": "ToAward",
    "type": "event"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "to",
        "type": "address"
      },
      {
        "internalType": "bytes32",
        "name": "hashvalue",
        "type": "bytes32"
      }
    ],
    "name": "emitDegree",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "account",
        "type": "address"
      }
    ],
    "name": "getDegree",
    "outputs": [
      {
        "internalType": "bytes32",
        "name": "",
        "type": "bytes32"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "string",
        "name": "_json",
        "type": "string"
      }
    ],
    "name": "hash",
    "outputs": [
      {
        "internalType": "bytes32",
        "name": "",
        "type": "bytes32"
      }
    ],
    "stateMutability": "pure",
    "type": "function"
  },
  {
    "inputs": [],
    "name": "owner",
    "outputs": [
      {
        "internalType": "address",
        "name": "",
        "type": "address"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  }
];

const address_founded = ref(null);
const address_searched = ref();


const searchDegree = async () => {
  console.log(address_searched.value);
  
  let contract = new web3.eth.Contract(abi, address);
  try {
    let degree_student = await contract.methods.getDegree(address_searched.value).call();
    if (degree_student != default_value_wallet) {
      address_founded.value = degree_student;
    } else {
      Swal.fire({
        title: "Não foi encontrada transação para esse endereço",
        icon: "error"
      }).then(() => {
        address_searched.value = null;
      });
    }
  } catch (error) {
    console.log(error);
  }
};

onMounted(async () => {



  /*
  Para enviar ao testnet sepolia
  const abi = [{ "inputs": [], "stateMutability": "nonpayable", "type": "constructor" }, { "anonymous": false, "inputs": [{ "indexed": true, "internalType": "address", "name": "_from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "_to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "Transfer", "type": "event" }, { "inputs": [{ "internalType": "address", "name": "account", "type": "address" }], "name": "balanceOf", "outputs": [{ "internalType": "uint256", "name": "", "type": "uint256" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "name", "outputs": [{ "internalType": "string", "name": "", "type": "string" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "owner", "outputs": [{ "internalType": "address", "name": "", "type": "address" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "symbol", "outputs": [{ "internalType": "string", "name": "", "type": "string" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "totalSupply", "outputs": [{ "internalType": "uint256", "name": "", "type": "uint256" }], "stateMutability": "view", "type": "function" }, { "inputs": [{ "internalType": "address", "name": "to", "type": "address" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }], "name": "transfer", "outputs": [], "stateMutability": "nonpayable", "type": "function" }];
  const web3 = new Web3(window.ethereum);
  const address = "0x30930a78aeB33BA3Ddf8995677e08703BF04a069";

  try {
    // Request account access if needed
    let sender = await window.ethereum.enable();
    
    let receiver = web3.eth.accounts.create();

    let contract = new web3.eth.Contract(abi, address);
    //let nome = await contract.methods.transfer("0x68d255d5968fe4910873815dd79c4216ac8b6df3",0.00000000001).call();
    let envio = contract.methods.transfer(receiver.address,20).send({
      from: sender[0]
    });
    
    console.log(envio);
    
    //console.log("Conta Secundaria: ",connection[0]);
    //console.log("Conta Secundaria: ",connection[1]);

    contract.methods.balanceOf(connection[0]).call().then(function (balance) {
      console.log(balance);
    });
  } catch (error) {
    // User denied account access...
  }
  */
});
</script>

<style scoped>
main {
  width: 100vw;
  height: 100vh;
  display: grid;
  grid-template-columns: 280px auto;
}

aside {
  height: 100vh;
}

section {
  width: auto;
  height: 100vh;
  padding: 5px;
}

.header {
  height: 10%;
  background-color: #1A1D20;
}

.content {
  height: 90%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #1A1D20;
}

.form {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
  background-color: #343A40;
  width: 80%;
}
</style>