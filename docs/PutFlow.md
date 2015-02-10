
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

    MaidClient::Put(D) | MaidManager<Client.name>::HandlePut(D) | DataManager<D.name>::HandlePut(D) | PmidManager<Pmid.name>::HandlePut(D) | PmidNode::HandlePut(D)


Implementation:

    MaidClient::Put(D) { MaidManager<Client.name>::HandlePut(D) }
    |
    MaidManager<Client.name>::HandlePut(D) {
      Allow ? [ ReserveCost(K*D.size()), DataManager<D.name>::HandlePut(D) ]
            : [ MaidClient::HandleOutOfCredit ]
    }
    |
    DataManager<D.name>::HandlePut(D) {
      [!Exist(D) ? Loop PmidNode in KClosestNodesTo(D.name)
                       PmidManager{PmidNode.name}::HandlePut(D) ]
    }
    |
    PmidManager<PmidNode.name>::HandlePut(D) {
      [ Account.Add(D), PmidNode::HandlePut(D) ]
    }
    |
    | PmidNode::HandlePut(D) {
        [!Store(D) ? PmidManager<PmidNode.name>::HandleStoreFailure(D) ]
    }

    < PmidNode::Persist(D) {/* resuming from above */}
    | PmidManager{PmidNode}::ConfirmStoragePromise(D.name) {
        Register(PmidNode, D.name)
        return Flow [ DataManager{D.name}, PersistedInVault(D.name,
                                                            Pmid.name) ] }
    | DataManager{D.name}::PersistedInVault(D.name, Pmid.name) {
        RegisterPmid(D.name, D.size, Pmid.name)
            // size known from D stored in InstantiateRegister
        InstantiateRegister.RemovePmid(Pmid.name)
        RedundantCopies = PmidsRegistered(D.name).size
        if (RedundantCopies >= 2) {
          MoveToLRUcache(InstantiateRegister(D.name).getData())
        }
        return Flow [ MaidManager{Maid}, AccountCost(Maid,
                            D.name, RedundantCopies * D.size) ] }
    | MaidManager{Maid}::AccountCost(Maid, D.name, cost) {
        PayInSafeCoinUnpaidRemainder(UpdatedCost = cost)
        return Flow [ MaidNode, PutConfirm(D.name, cost) ] }
    | MaidNode::PutConfirm(D.name, cost) {
        Return to higher levels, persistance can be deduced from cost }
    >

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
