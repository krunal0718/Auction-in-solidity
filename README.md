# Auction-in-solidity
Auction program in solidity language

// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0  <0.9.0;

contract auction {

    address payable public auticonner;

    uint public Starttime;
    uint public endtime;

    uint public hightbid;
    uint public highpayablebid;
    uint public incrementbid;



    enum  status{
        started,
        Running,
        ending,
        cancelled}
    
    status public auc_status;

    address payable   public hightbidder;

    mapping (address=>uint) public bids;

    constructor(){
        auticonner= payable(msg.sender);
        auc_status= status.Running; 
        Starttime =block.number;
        endtime= Starttime + 240;
        incrementbid = 1 ether;
    }

     modifier notowner(){
        require(msg.sender != auticonner, "onwer can not bid");
        _;
     }
     modifier owner(){
        require(msg.sender == auticonner, "onwer can not bid");
        _;
     }
     modifier started(){
        require(block.number > Starttime);
        _;
     }
      modifier beforeendeed(){
        require(block.number < endtime);
        _;
     }

     function andAuc() public owner{
        auc_status= status.ending;
     }

     function cancellAuc() public owner{
        auc_status= status.cancelled;
     }

     function min(uint a,uint b)  pure private returns(uint){

         if(a<=b)
         return a;
         else 
         return b;
     } 

    function bid() payable public notowner started beforeendeed {
       require(auc_status == status.Running);
       require(msg.value == 1 ether);

       uint currentbid = bids[msg.sender] + msg.value;

       require(currentbid >highpayablebid);

       bids[msg.sender] = currentbid;

       if(currentbid < bids[hightbidder]){

          highpayablebid = min(currentbid + incrementbid,  bids[hightbidder]);
       }
       else{
            highpayablebid = min(currentbid,bids[hightbidder]+incrementbid);
            hightbidder = payable(msg.sender);
       
       } 

    }

    function finalAug() view public {
        require(auc_status == status.cancelled || auc_status == status.ending || block.number> endtime);
        require(msg.sender == auticonner || bids[msg.sender]>0);

        address payable person;
        uint value;


        if(auc_status == status.cancelled)
        {
           person=payable(msg.sender);
           value=bids[msg.sender];
        }
        else{
          if(msg.sender == auticonner){
             person = auticonner;
             value = highpayablebid;
          }
          else{
             if( msg.sender == hightbidder ){
                 person= hightbidder;
                 value = bids[hightbidder]- highpayablebid;
             }
             else{
                 person = payable (msg.sender);
                 value = bids[msg.sender];
             }
          }

        }
        
        
    }

    function ResetBids() public{
            bids[msg.sender]=0;
        }     

    function transferEther(address payable person, uint256 value) public payable {
         person.transfer(value);
   }
}
