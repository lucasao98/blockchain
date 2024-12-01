<template>
    <main>
        <aside>
            <Sidebar menu="1" />
        </aside>
        <section class="bg-dark bg-gradient">
            <div class="header text-white d-flex align-items-center p-3 mb-1 rounded">
                <h1>Cadastrar Diploma</h1>
            </div>
            <div class="content">
                <div class="form p-4 rounded">
                    <div class="info-address d-flex gap-4">
                        <div class="input-group mt-5">
                            <span class="input-group-text" id="inputGroup-sizing-default">Endereço da instituição</span>
                            <input type="text" class="form-control" aria-label="Sizing example input"
                                aria-describedby="inputGroup-sizing-default" readonly :value="address_from">
                        </div>
                        <div class="input-group mt-5">
                            <span class="input-group-text" id="inputGroup-sizing-default">Endereço do Aluno</span>
                            <input type="text" class="form-control" aria-label="Sizing example input"
                                aria-describedby="inputGroup-sizing-default" v-model="address_to"
                                @focusout="verifyHasDegree()">
                        </div>
                    </div>

                    <hr class="border border-secondary border-2">

                    <div class="input-group info-student">
                        <div class="mb-3 text-white">
                            <label for="formGroupExampleInput" class="form-label">Nome da Instituição</label>
                            <input type="text" class="form-control" id="formGroupExampleInput" v-model="institution" />
                        </div>
                        <div class="mb-3 text-white">
                            <label for="formGroupExampleInput2" class="form-label">Nome Completo do Aluno</label>
                            <input type="text" class="form-control" id="formGroupExampleInput2" v-model="full_name" />
                        </div>
                        <div class="mb-3 text-white">
                            <label for="formGroupExampleInput2" class="form-label">Nome do Curso</label>
                            <input type="text" class="form-control" id="formGroupExampleInput2" v-model="course_name" />
                        </div>
                        <div class="mb-3 text-white">
                            <label for="formGroupExampleInput2" class="form-label">Data de Conclusão</label>
                            <input type="date" class="form-control" id="formGroupExampleInput2"
                                v-model="conclusion_date" />
                        </div>

                        <button type="button" @click="registerDegree()"
                            class="btn btn-primary w-25 rounded">Registrar</button>
                    </div>
                </div>
            </div>
        </section>
    </main>
</template>

<script setup>
import Web3 from 'web3';
import { ref, onMounted, watch } from 'vue';
import Sidebar from '@/components/SideBar.vue';
import Swal from 'sweetalert2';

const address_from = ref();
const address_to = ref();
const institution = ref();
const full_name = ref();
const course_name = ref();
const conclusion_date = ref();
const data_in_json = { institution: null, full_name: null, course_name: null, conclusion_date: null };
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

const verifyHasDegree = async () => {
    let contract = new web3.eth.Contract(abi, address);
    try {
        let degree_student = await contract.methods.getDegree(address_to.value).call();
        if (degree_student != default_value_wallet) {
            Swal.fire({
                title: "Aluno já possui um diploma",
                icon: "error"
            }).then(() => {
                address_to.value = null;
            });
        } else if (address_to.value == address_from.value) {
            Swal.fire({
                title: "Não é possível enviar um diploma para esse endereço!",
                icon: "error"
            }).then(() => {
                address_to.value = null;
            });
        }
    } catch (error) {
        console.log(error);
    }
};

const registerDegree = async () => {
    let contract = new web3.eth.Contract(abi, address);

    data_in_json.institution = institution.value;
    data_in_json.course_name = course_name.value;
    data_in_json.full_name = full_name.value;
    data_in_json.conclusion_date = conclusion_date.value;

    try {
        let string_data = JSON.stringify(data_in_json);
        let hashed_json = await contract.methods.hash(string_data).call();

        let transaction_register = await contract.methods.emitDegree(address_to.value, hashed_json).send({ from: address_from.value });
        Swal.fire({
            title: "Transação Cadastrada com Sucesso!",
            icon: "success"
        }).then(() => {
            institution.value = null;
            course_name.value = null;
            full_name.value = null;
            conclusion_date.value = null;
            address_to.value = null;
        });
        //let degree_student = await contract.methods.getDegree(address_to.value).call();
        //console.log(degree_student);
    } catch (error) {
        console.log(error);
    }



};

onMounted(async () => {
    try {
        const accounts = await web3.eth.getAccounts();
        address_from.value = accounts[0];
        const conta1 = accounts[0];
        const conta2 = accounts[1];
        web3.eth.defaultAccount = conta1;


        //let json = { name: "Lucas", course: "Biologia", year_formation: "2022", president_name: "Alessandro" };
        //let to_string = JSON.stringify(json);

        //let hash_criado = await contract.methods.hash(to_string).call();
        //console.log("hash criado: ", hash_criado);

        // let emissao = await contract.methods.emitDegree(conta2, hash_criado).send({
        //     from: conta1
        // });

        // let a = await contract.methods.getDegree(conta2).call();
        // console.log("valor da conta 2", a);

    } catch (error) {
        console.log(error);

    }
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

.info-student {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
}
</style>