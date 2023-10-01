### Teknologi Blockchain

Nama : Muhammad Faris Anwari<br>
NRP : 5027201008
<br>
Tugas :
1. Buatkan endpoint yang mengecek apakah chainnya valid
2. Simpan hash block didalam blocknya
3. Tambah data diddalam block
4. Fungsi dan endpoint yang mensimulasikan adanya modifikasi didalam block, sehingga chainnya tidak valid

#### 1. Check chain validity
```py
def is_chain_valid(self):
        prev_block = self.chain[0]
        now_index = 1
        while now_index < len(self.chain):
            now_block = self.chain[now_index]

            # 1. cek apakah hash now block = prev hash 
            # prev_hash = self.get_hash(prev_block)
            prev_hash = prev_block['hash']
            if prev_hash != now_block['prev_hash']:
                return False

            # 2. cek apakah proof of worknya true
            now_proof = now_block['proof']
            prev_proof = prev_block['proof']
            hash_operation = hashlib.sha256(str(now_proof**2 - prev_proof**2).encode()).hexdigest()
            if hash_operation[:4] != '0000':
                return False

            prev_block = now_block
            now_index += 1
        return True
```
```py
@app.route("/check_valid", methods=['GET'])
def check_valid():
    valid = blockchain.is_chain_valid()
    response = {
        'validity': "Blockchain is " + str(valid)
    }
    return jsonify(response), 200
```

#### 2. Store block hash within said block
```py
def __init__(self):
        self.chain = []
        # genesis block
        self.create_block(proof = 1, prev_hash = None, curr_hash= None, proof_of_work = None, data = None)

    def create_block(self, proof, prev_hash, curr_hash, proof_of_work, data):
        
        block = {
            'id_block': len(self.chain) + 1,
            'timestamp': str(datetime.datetime.now()),
            'proof': proof,
            'prev_hash': prev_hash,
            'hash': curr_hash,
            'proof_of_work': proof_of_work,
            'data': data
        }
        
        self.chain.append(block)
        return block
```
```py
@app.route("/mining", methods=['GET'])
def mining():
    prev_block = blockchain.get_last_block()
    # get prev proof 
    prev_proof = prev_block['proof']
    proof, proof_of_work = blockchain.proof_of_work(prev_proof) 
    # get prev hash 
    # prev_hash = blockchain.get_hash(prev_block)
    prev_hash = prev_block['hash']
    # set curr_hash value to None
    curr_hash = None
    # create block 
    created_block = blockchain.create_block(proof, prev_hash, curr_hash, proof_of_work)
    
    # get curr hash
    curr_block = blockchain.get_last_block()
    curr_hash = blockchain.get_hash(curr_block)

    curr_block['hash'] = curr_hash

    response = {
        'message': "Blockchain is successfully created",
        'created block': created_block,
        'current hash': curr_hash
    }
    return jsonify(response), 200
```
#### 3. Add data in block
```py
@app.route("/create", methods=['POST'])
def create_block():
    data = request.form

    prev_block = blockchain.get_last_block()
    # get prev proof 
    prev_proof = prev_block['proof']
    proof, proof_of_work = blockchain.proof_of_work(prev_proof) 
    # get prev hash 
    # prev_hash = blockchain.get_hash(prev_block)
    prev_hash = prev_block['hash']

    curr_hash = None
    # create block 
    created_block = blockchain.create_block(proof, prev_hash, curr_hash, proof_of_work, data['data'])

    # get curr hash
    curr_block = blockchain.get_last_block()
    curr_hash = blockchain.get_hash(curr_block)

    curr_block['hash'] = curr_hash
    response = {
        'message': "Blockchain is successfully created",
        'created block': created_block
    }
    return jsonify(response), 200
```
#### 4. Ruin chain
```py
@app.route("/ruin_chain", methods=['GET'])
def ruin_chain():
    curr_block = blockchain.get_last_block()
    curr_block['prev_hash'] = curr_block['hash']
    response = {
        'message': "Blockchain has been ruined",
        'block': curr_block
    }
    return jsonify(response), 200
```