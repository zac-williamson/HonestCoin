HonestCoin, the 100% Honest and Fair ICO
HonestCoin is the hottest new ICO contract that issues tokens in exchange for ether (1:1 ratio for simplicity)

Because the creators of HonestCoin care about transparency, they gaurantee their contract has the following features:

## Deposited Ether can only be claimed after 6 months
There is a build in check which ensures that the withdrawFromEscrow() function can only be called after 6 months worth of blocks have elapsed since contract creation. This ensures creators have an incentive to, you know, create.
## For the first 5 months, users can vote on whether the 'freeze' the ICO.
If the vote to freeze passes, users can prevent ICO creators from withdrawing ether! Users can even withdraw their own initial ether once the contract is frozen. Clearly with such a contract design, the ICO creators have every incentive to keep their investors happy!
## Users can even transfer the ICO to another person
The HonestCoin creators are so *confident* that they will deliver, they have built in a transfer function! Anybody can initiate a vote to transfer ownership of the ICO to a nominated address. Users can then vote to voice their approval. If the total votes > half the ICO balance, ICO ownership is transferred to the new address. This ensures that the ICO is controlled by the *worthiest* individual!
## The ICO will have two rounds, each lasting for a week
This is hard-coded into the HonestCoin smart contract, ensuring the ICO owners cannot continue opening up ICO rounds, diluting previous investors.
## After 6 months

Imagine the following case...
# HonestCoin is a *tremendous* success, raising millions of dollars!
Investors are enthusiastic! However, as time passes concern rises that the developers are failing to deliver on thier promises and are more concerned with fact-fining trips to Barbados. Calls to freeze the ICO grow and soon over half of the investors have voted to freeze! But...there's a bug!

```
    function resolveFreezeVote() {
        if (block.number > creationBlock + validationDuration) {
            uint256 freezeVotes = 0;
            uint256 proceedVotes = 0;
            for (var i = 0; i < votes.length; i++) {
                if (votes[i].votesToFreeze) {
                    freezeVotes += balances[votes[i].voteAddress];
                } else {
                    proceedVotes += balances[votes[i].voteAddress];
                }
            }
            if (freezeVotes > proceedVotes) {
                frozen = true;
            }
        }
    }
```

The votes are tallied by iterating over the 'votes' array. However as so many investors have earnestly voted, this array is so large that the gas cost to run the function exceeds the block gas limit! The ICO contract can't be frozen!

## The community is so OUTRAGED at this trick, miners vote to increase the block gas limit *just* so resolveFreezeVote() can be run, but it has no effect
In the 'for' loop, 'i' is initialized as a variant set to '0'. 0's type is an 8-bit character byte, ensuring that i cannot exceed 256. As long as votes.length > 256, the for loop is infinite and cannot be run. 
Of course, the HonestCoin creators, being the honest folk they are, interfered so that the votes array was 256 entries large by mining the block which created HonestCoin. They ensured that the rest of the block transaction list was filled up with function calls to add to the 'votes' array through HonestCoin::castFreezeVote.

## Fearing the worst, the community organizes around a group of white hats offering to take control of HonestCoin. They begin the transfer ICO process
HonestCoin, being honest chaps, have an in-built transfer mechanism in their contract. Anyone can call HonestCoin::initiateTransferVote to create a ChangeControllerContract and nominate a new controller. HonestCoin holders can then vote to approve of the new controller. If the total number of votes for the new controller is over half of HonestCoin's token balance, a new controller is designated!

```
    mapping (address => bool) transferVotes;
    function initiateTransferVote(address newController) {
        ChangeControllerContract changeControllerContract = new ChangeControllerContract(newController);
        transferVotes[changeControllerContract] = true;
    }

    function transferOwnership(address newController) {
        if (transferVotes[msg.sender]) {
            owner = newController;
        }
    }
//--------
contract ChangeControllerContract
{
    HonestCoin honestContract;
    uint256 endBlock;
    address potentialController;

    mapping (address => uint256) userVotes;
    uint256 totalVotes;

    function ChangeControllerContract(address newController) {
        this.initialise(msg.sender, newController); //'this' is always null in a contructor because the contract has not been added to the blockchain.
                                                    // the constructor will always throw an exception, so a ChangeControllerContract cannot be created.
    }

    function initialise(address _sender, address _newController) {
        honestContract = HonestCoin(_sender);
        endBlock = block.number + 60480; //one week
        potentialController = _newController;
    }

    function castVote() {
        totalVotes += userVotes[msg.sender] - honestContract.balanceOf(msg.sender);
        userVotes[msg.sender] = honestContract.balanceOf(msg.sender);
    }

    function resolve() {
        if (block.number > endBlock) {
            if (totalVotes > honestContract.totalSupply() / 2) {
                honestContract.transferOwnership(potentialController);
            }
        }
    }
}
```

However, there's a bug! To the white hat's horror, ChangeControllerContract cannot be created!
