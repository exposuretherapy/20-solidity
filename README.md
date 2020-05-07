# 20-solidity
Creating deployable smart contract for payments

### Associate Profit Splitter:
pragma solidity ^0.5.0;

// lvl 1: equal split
contract AssociateProfitSplitter {
    // @TODO: Create three payable addresses representing `employee_one`, `employee_two` and `employee_three`.
    address payable public employee_one;
    address payable public employee_two;
    address payable public employee_three;
    
    constructor(address payable _one, address payable _two, address payable _three) public {
        employee_one = _one;
        employee_two = _two;
        employee_three = _three;
    }

    function balance() public view returns(uint) {
        return address(this).balance;
    }

    function deposit() public payable {
        // @TODO: Split `msg.value` into three
        uint amount = msg.value/3; // Your code here!
        

        // @TODO: Transfer the amount to each employee
        employee_one.transfer(amount);
        employee_two.transfer(amount);
        employee_three.transfer(amount);
        

        // @TODO: take care of a potential remainder by sending back to HR (`msg.sender`)
        msg.sender.transfer(msg.value - amount*3);
    }

    function() external payable {
        // @TODO: Enforce that the `deposit` function is called in the fallback function!
      deposit();
    }
}


### Tiered Profit Spliiter
pragma solidity ^0.5.0;

// lvl 2: tiered split
contract TieredProfitSplitter {
    address payable employee_one; // ceo
    address payable employee_two; // cto
    address payable employee_three; // bob

    constructor(address payable _one, address payable _two, address payable _three) public {
        employee_one = _one;
        employee_two = _two;
        employee_three = _three;
    }

    // Should always return 0! Use this to test your `deposit` function's logic
    function balance() public view returns(uint) {
        return address(this).balance;
    }

    function deposit() public payable {
        uint points = msg.value / 100; // Calculates rudimentary percentage by dividing msg.value into 100 units
        uint total;
        uint amount;
        
        amount = points*60;
        total += amount;
        employee_one.transfer(amount);
        
        amount = points*30;
        total += amount;
        employee_two.transfer(amount);
        
        amount = points*100;
        total += amount;
        employee_three.transfer(amount);

        employee_one.transfer(msg.value - total); // ceo gets the remaining wei
    }

    function() external payable {
        deposit();
    }
}

### Deferred Equity Plan
pragma solidity ^0.5.0;

contract DeferredEquityPlan {
    address human_resources;
    
    address payable employee; //bob
    bool active = true; //this employee is active at the start of the contract.
    
    //Set the total shares and annual distribution
    uint public distributed_shares;
    uint public total_shares = 1000;
    uint public annual_distribution = 250;
    
    
    //Set the unlock_time to be 365 days from now.
    uint public start_time = now;
    uint public unlock_time;    
    
    
    constructor(address payable _employee) public {
        human_resources = msg.sender;
        employee = _employee;
    }
    
    function distribute() public {
        require(msg.sender == human_resources || msg.sender == employee, "You are not authorized on this contract.");
        require(active == true, "Contract not active.");
        
        //Add require unlock time less than or equal to now
        require(unlock_time <= start_time, "You need to wait one year");
        
        //Add require distributed shares is less than total shares
        require(address(this).balance < total_shares, "You don't have enough shares.");
        
        
        //Add 365 days to unlock time
        unlock_time = now + 365 days;

        /*
        Calculate the shares distributed by using the function:
            (now-start_time)/365 * annual distribution
            
        Double check in case the employee doesnt cash out until after 5+ years
            if (distributed_shares>1000) {
                distributed_shares=1000;
            }
        */
        
        distributed_shares = (now-start_time)/365 * annual_distribution;
        if (distributed_shares>total_shares) {
            distributed_shares=total_shares;
        }
        
    }
    
        //hr and employee can deactivate contract at will
    function deactivate() public {
        require(msg.sender == human_resources || msg.sender == employee, "You are not authorized to deactivate this contract.");
        active = false;
        
    }
    
    function () external payable {
        revert("Don't Send Ether to this contract");
    }
}