LNCToken
---------------
https://etherscan.io/address/0x63e634330a20150dbb61b15648bc73855d6ccf07#code

Integer Overflow


    /// allows to transfer token to another address
    function transfer(address _to, uint256 _value) returns (bool success) {
        // Don't allow in funding state
        if(funding) throw;
        if(!allowTransfer)throw;
    
        var senderBalance = balances[msg.sender];
        //only allow if the balance of the sender is more than he want's to send
        if (senderBalance >= _value && _value > 0) {
            //reduce the sender balance by the amount he sends
            senderBalance -= _value;
            balances[msg.sender] = senderBalance;
    
            //increase the balance of the receiver by the amount we reduced the balance of the sender
            balances[_to] += _value;
    
            //saves the last time someone sent LNc from this address
            //is needed for our Token Holder Tribunal
            //this ensures that everyone can only vote one time
            //otherwise it would be possible to send the LNC around and everyone votes again and again
            lastTransferred[msg.sender]=block.timestamp;
            Transfer(msg.sender, _to, _value);
            return true;
        }
        //transfer failed
        return false;
    }


If calls this function with a large _value, which is smaller than the senderBalance but the sum of (balances\[\_to\]+\_value) > 2^256, it will cause an integer overflow at line ('balances[\_to] += _value;') and finally change the balance of receiver's accounts to a smaller number(caused by overflow). This integer overflow vulnerability allows sender to cause unexpected economic losses.

The founder should check the sum of (balances\[\_to\]+\_value) before changing the balances of sender and receiver, such as 'if (balances[_to] < balances[\_to] + \_value) throw;'.

