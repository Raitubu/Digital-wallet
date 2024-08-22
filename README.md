# Digital-wallet C++
// Digiwallet blockchain.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <vector>
#include <ctime>
#include <openssl/sha.h>
#include <iomanip>
#include <sstream>

// Function to compute SHA-256 hash
std::string sha256(const std::string& data) {
    unsigned char hash[SHA256_DIGEST_LENGTH];
    SHA256(reinterpret_cast<const unsigned char*>(data.c_str()), data.size(), hash);
    std::stringstream ss;
    for (int i = 0; i < SHA256_DIGEST_LENGTH; ++i) {
        ss << std::hex << std::setw(2) << std::setfill('0') << static_cast<int>(hash[i]);
    }
    return ss.str();
}

// Basic Wallet Class
class Wallet {
private:
    std::string publicKey;
    std::string privateKey;

    std::string generatePrivateKey() {
        // This is a placeholder; use a proper cryptographic key generation method in real applications
        return sha256(std::to_string(std::time(0)));
    }

public:
    Wallet() {
        privateKey = generatePrivateKey();
        publicKey = sha256(privateKey);  // Simplified public key derived from private key
    }

    std::string getPublicKey() const {
        return publicKey;
    }

    std::string getPrivateKey() const {
        return privateKey;
    }
};

// Transaction structure
struct Transaction {
    std::string sender;
    std::string recipient;
    double amount;

    std::string toString() const {
        return sender + recipient + std::to_string(amount);
    }
};

// Block structure
struct Block {
    int index;
    std::string previousHash;
    std::string timestamp;
    std::vector<Transaction> transactions;
    std::string hash;

    Block(int idx, std::string prevHash, std::string ts, std::vector<Transaction> txs)
        : index(idx), previousHash(prevHash), timestamp(ts), transactions(txs) {
        hash = calculateHash();
    }

    std::string calculateHash() const {
        std::string txData;
        for (const auto& tx : transactions) {
            txData += tx.toString();
        }
        return sha256(std::to_string(index) + previousHash + timestamp + txData);
    }
};

// Blockchain class
class Blockchain {
private:
    std::vector<Block> chain;
    std::vector<Transaction> currentTransactions;

public:
    Blockchain() {
        // Create the genesis block
        chain.emplace_back(0, "0", "2024-08-14", std::vector<Transaction>{});
    }

    Block getLatestBlock() const {
        return chain.back();
    }

    void addTransaction(const Transaction& tx) {
        currentTransactions.push_back(tx);
    }

    void addBlock() {
        Block newBlock(chain.size(), getLatestBlock().hash, "2024-08-14", currentTransactions);
        chain.push_back(newBlock);
        currentTransactions.clear();
    }

    void printChain() const {
        for (const auto& block : chain) {
            std::cout << "Index: " << block.index << "\n"
                << "Previous Hash: " << block.previousHash << "\n"
                << "Timestamp: " << block.timestamp << "\n"
                << "Hash: " << block.hash << "\n"
                << "Transactions:\n";

            for (const auto& tx : block.transactions) {
                std::cout << "  Sender: " << tx.sender << ", Recipient: " << tx.recipient << ", Amount: " << tx.amount << "\n";
            }
            std::cout << "\n";
        }
    }
};

int main() {
    Blockchain myBlockchain;
    Wallet wallet1;
    Wallet wallet2;

    // Create transactions
    Transaction tx1{ wallet1.getPublicKey(), wallet2.getPublicKey(), 100.0 };
    Transaction tx2{ wallet2.getPublicKey(), wallet1.getPublicKey(), 50.0 };

    // Add transactions to blockchain
    myBlockchain.addTransaction(tx1);
    myBlockchain.addTransaction(tx2);
    myBlockchain.addBlock();

    myBlockchain.printChain();

    return 0;



  
  

   
