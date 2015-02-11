
# MaidSafe Language of the Network


The network allows

legend:

    |      sentinel
    D      data
    Manager<Address>
    ->> send to multiple non-NAE
    -> send to one non-NAE
    Loop x in xs : 


### MAID PUT
all functions are templated on DataType

    MaidClient::Put(D) | MaidManager<Client.name>::HandlePut(D)
                       | DataManager<D.name>::HandlePut(D)
                       ->> | PmidManager<Pmid.name>::HandlePut(D)
                       | PmidNode::HandlePut(D)


Implementation:

    MaidClient::Put(D) { MaidManager<Client.name>::HandlePut(D) }

    MaidManager<Client.name>::HandlePut(D) {
      Allow ? [ ReserveCost(K*D.size()), DataManager<D.name>::HandlePut(D) ]
            : [ MaidClient::HandleOutOfCredit ]
    }

    DataManager<D.name>::HandlePut(D) {
      !Exist(D) ? (Account.Create(D))
                  (Loop PmidNode in KClosestNodesTo(D.name) :
                     [ D.Account.Add(Pmid), PmidManager<PmidNode.name>::HandlePut(D) ])
    }
    
    DataManager<D.name>::HandlePutFailure(D, Pmid) {
      EXIST(D) ? (D.Account.Remove(Pmid))
                 (D.Accout.Pmids.Count < ENOUGH
                      ? (NewPmid = GetNewPmid())
                        ([ D.Account.Add(NewPmid), PmidManager<NewPmid.name>::HandlePut(D) ])
      DownRank(Pmid)   
    }
    
    PmidManager<PmidNode.name>::HandlePut(D) {
      [ PmidNode.Account.Add(D), PmidNode::HandlePut(D) ]
    }
    
    PmidManager<PmidNode.name>::HandlePutFailure(D) {
      [Client.Account.Subtract(D), DataManager<D.name>::HandlePutFailure(D, Pmid)]
    }
    
    PmidNode::HandlePut(D) {
      [!Store(D) ? PmidManager<PmidNode.name>::HandlePutFailure(D) ]
    }

### MAID GET

    MaidClient::Get(D.name) | DataManager<D.name>::HandleGet(D.name)
                            ->> | PmidNode::HandleGet(D.name)
                            -> MaidClient::GetResponse(D)
    
    Implementation
    
    MaidClient::Get(D.name) { DataManager<D.name>::HandleGet(D.name) }
    
    DataManager<D.name>::HandleGet(D.name) {
      EXIST(D) ? [Loop PmidNode in D.Account.PmidNodes : PmidNode::HandleGet(D.name)]
               : MaidClient::HandleGetFailure(D.name)
    }


### CHURN HANDLE
