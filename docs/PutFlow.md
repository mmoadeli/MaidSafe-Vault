
# MaidSafe Language of the Network


The network allows

legend:

    <      scatter
    >      gather
    |      sentinel
    D      data
    H()    Hash512
    H^n()  n-th Hash512
    Manager{Address};
           {Address} omitted where evident,
           e.g. MaidManagers{MaidNode}


### MAID PUT
all functions are templated on DataType

MAID PUT and MAID PUT CONFIRM

    MaidClient::Put(D) | MaidManager<Client.name>::HandlePut(D)
                       | DataManager<D.name>::HandlePut(D)
                       | PmidManager<Pmid.name>::HandlePut(D)
                       | PmidNode::HandlePut(D)


Implementation:

    MaidClient::Put(D) { MaidManager<Client.name>::HandlePut(D) }

    MaidManager<Client.name>::HandlePut(D) {
      Allow ? [ ReserveCost(K*D.size()), DataManager<D.name>::HandlePut(D) ]
            : [ MaidClient::HandleOutOfCredit ]
    }

    DataManager<D.name>::HandlePut(D) {
      !Exist(D) ? (Account.Create(D))
                  (Loop PmidNode in KClosestNodesTo(D.name)
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

    < MaidNode::Get(D.name) {
        client_routing.get(D.name) }
    | DataManager{D.name}::HandleGet(D.name, ReplyToClient) {
        if (LRUcache.get(D.name)) {
          return Flow [ ReplyToClient, GetResponse(LRUcache.get(D.name)) ]
        }
        OnlinePmidNodes = Register.getOnlinePmidNodes(D.name)
        return Flow [ OnlinePmidNodes, PushForward(D.name, ReplyToClient) ] }
    | PmidNode{OnlinePmidNode}::PushForward(D.name, ReplyToClient) {
        return Flow [ ReplyToClient, GetResponse(Vault.get(D.name)) }
    >

### CHURN HANDLE
