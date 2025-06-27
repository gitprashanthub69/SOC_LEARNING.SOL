
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
}
