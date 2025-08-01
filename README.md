
#  Blockchain Crowdfunding Project

**Name**: Prashant  
**Roll No**: 24B2158  

---

##  Summary: What I’ve Learned So Far

###  Week 1: Introduction to Blockchain & Ethereum
- Set up MetaMask, Remix, and Hardhat
- Learned Solidity basics (contracts, variables)
- Deployed a simple “Hello World” contract

###  Week 2: Campaign Struct & Storage
- Created `Campaign` struct with fields: owner, title, goal, deadline
- Used `mapping(uint => Campaign)` to manage multiple campaigns
- Built `createCampaign()` to store campaigns

###  Week 3: Contribution Logic
- Added `contribute()` function to accept ETH from users
- Tracked total raised amount and individual contributions
- Used `msg.sender` and `msg.value` for contributor tracking

###  Week 4: Funding & Refund Logic
- Added `releaseFunds()` to transfer money to owner if goal met
- Added `refund()` for contributors if goal not met
- Applied `require`, `block.timestamp`, and `modifiers` for validation

---

## 🧾 Code (Merged Contract Week 1–4)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Crowdfunding {
    struct Campaign {
        address payable owner;
        string title;
        uint goal;
        uint deadline;
        uint raisedAmount;
        bool completed;
        mapping(address => uint) contributions;
    }

    uint public campaignCount;
    mapping(uint => Campaign) private campaigns;

    // Week 1: Basic Hello World logic can be interpreted as contract setup
    string public message = "Hello Blockchain World!";

    // Week 2: Create campaign
    function createCampaign(string memory _title, uint _goal, uint _durationInSeconds) public {
        campaignCount++;
        Campaign storage c = campaigns[campaignCount];
        c.owner = payable(msg.sender);
        c.title = _title;
        c.goal = _goal;
        c.deadline = block.timestamp + _durationInSeconds;
        c.raisedAmount = 0;
        c.completed = false;
    }

    // Week 3: Contribute ETH to campaign
    function contribute(uint _id) public payable {
        Campaign storage c = campaigns[_id];
        require(block.timestamp < c.deadline, "Campaign has ended");
        require(msg.value > 0, "Must send ETH");

        c.raisedAmount += msg.value;
        c.contributions[msg.sender] += msg.value;
    }

    function getContribution(uint _id, address _user) public view returns (uint) {
        return campaigns[_id].contributions[_user];
    }

    // Week 4: Fund release and refund logic
    function releaseFunds(uint _id) public {
        Campaign storage c = campaigns[_id];
        require(block.timestamp >= c.deadline, "Campaign still active");
        require(c.raisedAmount >= c.goal, "Goal not met");
        require(!c.completed, "Funds already released");

        c.completed = true;
        c.owner.transfer(c.raisedAmount);
    }

    function refund(uint _id) public {
        Campaign storage c = campaigns[_id];
        require(block.timestamp >= c.deadline, "Campaign still active");
        require(c.raisedAmount < c.goal, "Goal was met");

        uint amount = c.contributions[msg.sender];
        require(amount > 0, "No contribution to refund");

        c.contributions[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

    function getCampaign(uint _id) public view returns (
        address owner,
        string memory title,
        uint goal,
        uint deadline,
        uint raisedAmount,
        bool completed
    ) {
        Campaign storage c = campaigns[_id];
        return (c.owner, c.title, c.goal, c.deadline, c.raisedAmount, c.completed);
    }
)
// Crowdfunding DApp - Week 5 to Week 8 Code Breakdown


/*
----------------------------------
Week 5: Events & Smart Contract Security
----------------------------------
*/

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract SecureCrowdfund {
    uint public totalCampaigns;

    struct Campaign {
        address payable owner;
        string title;
        string description;
        uint goal;
        uint deadline;
        uint raised;
        bool withdrawn;
        mapping(address => uint) donations;
    }

    mapping(uint => Campaign) private campaigns;
    mapping(uint => address[]) private donors;

    event CampaignCreated(uint indexed id, address indexed owner, uint goal, uint deadline);
    event Donated(uint indexed id, address indexed donor, uint amount);
    event FundsWithdrawn(uint indexed id, uint amount);
    event Refunded(uint indexed id, address indexed donor, uint amount);

    modifier validCampaign(uint _id) {
        require(_id < totalCampaigns, "Invalid campaign ID");
        _;
    }

    modifier onlyOwner(uint _id) {
        require(msg.sender == campaigns[_id].owner, "Not campaign owner");
        _;
    }

    function createCampaign(string memory _title, string memory _description, uint _goal, uint _duration) public {
        require(_goal > 0, "Goal must be > 0");
        require(_duration > 0, "Duration must be > 0");

        Campaign storage c = campaigns[totalCampaigns];
        c.owner = payable(msg.sender);
        c.title = _title;
        c.description = _description;
        c.goal = _goal;
        c.deadline = block.timestamp + _duration;

        emit CampaignCreated(totalCampaigns, msg.sender, _goal, c.deadline);
        totalCampaigns++;
    }

    function donate(uint _id) public payable validCampaign(_id) {
        Campaign storage c = campaigns[_id];
        require(block.timestamp < c.deadline, "Campaign ended");
        require(msg.value > 0, "Must send ETH");

        if (c.donations[msg.sender] == 0) {
            donors[_id].push(msg.sender);
        }
        c.donations[msg.sender] += msg.value;
        c.raised += msg.value;

        emit Donated(_id, msg.sender, msg.value);
    }

    function withdrawFunds(uint _id) public validCampaign(_id) onlyOwner(_id) {
        Campaign storage c = campaigns[_id];
        require(block.timestamp >= c.deadline, "Campaign still active");
        require(c.raised >= c.goal, "Goal not met");
        require(!c.withdrawn, "Already withdrawn");

        c.withdrawn = true;
        c.owner.transfer(c.raised);

        emit FundsWithdrawn(_id, c.raised);
    }

    function refund(uint _id) public validCampaign(_id) {
        Campaign storage c = campaigns[_id];
        require(block.timestamp >= c.deadline, "Campaign still running");
        require(c.raised < c.goal, "Goal was met");

        uint donated = c.donations[msg.sender];
        require(donated > 0, "No donation to refund");

        c.donations[msg.sender] = 0;
        payable(msg.sender).transfer(donated);

        emit Refunded(_id, msg.sender, donated);
    }

    // View campaign summary
    function getCampaign(uint _id) public view validCampaign(_id) returns (
        address owner,
        string memory title,
        string memory description,
        uint goal,
        uint deadline,
        uint raised,
        bool withdrawn
    ) {
        Campaign storage c = campaigns[_id];
        return (c.owner, c.title, c.description, c.goal, c.deadline, c.raised, c.withdrawn);
    }

    // Donation amount by donor
    function donationOf(uint _id, address donor) public view validCampaign(_id) returns (uint) {
        return campaigns[_id].donations[donor];
    }

    // List of donor addresses
    function getDonors(uint _id) public view validCampaign(_id) returns (address[] memory) {
        return donors[_id];
    }
}



/*
----------------------------------
Week 6: Testing and Contract Polish
----------------------------------
*/

//  Test with different cases on Remix or Hardhat
//  Check for edge cases: double refund, early withdraw
//  Optimize for gas by removing unnecessary storage updates
//  Ensure clean and readable naming convention

/*
// PLACEHOLDER for tests (in Hardhat or Remix)

1. Create campaign
2. Donate before deadline
3. Attempt to donate after deadline (should fail)
4. Try early withdrawal (should fail)
5. Withdraw after goal met (success)
6. Refund donors if goal not met (success)
*/



/*
----------------------------------
Week 8: Final Wrap-Up
----------------------------------
*/

//  Deploy to a public testnet (Goerli, Sepolia)
// Share contract address with mentor
//  Optional: Write blog, demo, or record walkthrough
//  Upload repo to GitHub and link demo video

/*
Instructions to Deploy:
- Compile and deploy contract to Goerli using Remix or Hardhat
- Verify contract if using Etherscan
- Share deployed address and ABI with frontend if built
*/

