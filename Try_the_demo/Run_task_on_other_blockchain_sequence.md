```
@startuml
autonumber

box Task Client DApp browser
actor client
end box

box Substrate Demo Network 
participant alice
end box

box  Tea Network
participant layer1Alice
participant layer1Bob
participant layer1Charlie
end box

client->alice: request delegate
note right
request_delegate(sender, employer, net_address, fee)
record employer information to chain
   insert into EmployersApplys(block_number, vec[(employer, sender])
   insert into Employers(employer, false)
   insert into EmployerSender(employer, sender)
   insert into EmployerFee(empolyer, fee)
end note

alice-->layer1Alice: request delegate
note right
 request_delegate(employer, net_address) {
   Call BeMyDelegateRequest
 }
end note

layer1Alice-->alice: response of BeMyDelegateRequest
note left
save_delegate_info 
 DelegateInfo 
    - delegator_tea_id: Vec<u8>,
    - delegator_ephemeral_id: Vec<u8>,
    - sig: Vec<u8>,
    - key3_rsa_pub_key: String,
 save into local storate
end note

...

alice-[#blue]>alice: <OffChainWorker> apply delegate
rnote right
apply_delegates(block_number)
 update_delegate_status(employer, updater)
 need updater sign 
 insert into Employers(employer, true)
end note

client->alice: begin task
note right
begin_task(sender, employer, description_cid, fee)
 insert into Tasks(block_number, vec[TaskInfo])
 insert into EmployerTaskFee(employer, fee)
end note

alice-[#blue]>alice:<OffChanWorker> send errand tasks
rnote right
range tasks {
   send_task_to_tea_network(
            account: &AccountId32,
            description_cid: &Cid,
            errand_id: &ErrandId,
            net_address: &NetAddress,
        ) {
             get DelegateInfo from local storage: load_delegate_info(employer)
             send task to tea network
        }
   init_single_errand_task(
            signer: &Signer<T, T::AuthorityId, ForAll>,
            employer: &T::AccountId,
            description_cid: &Cid,
            errand_id: &ErrandId,
        ) {
               init_errand(employer, errand_id, description_cid) {
                   insert into ProcessingErrands(description_cid)
               }
        }
 }
end note

alice->layer1Alice: send task to tea network
alice-[#blue]>alice:<OffChainWorker> query errand task results
rnote right
query_errand_task_results(block_number)
  check has Alice account
  range ProcessingErrands {
      fetch_single_task_result {
           http_query_task_result(errand_id, net_address)
      }
 }
end note

alice-->layer1Alice: http query task result
layer1Alice-->alice: respons of task result
note left
get ErrandResultInfo from response
 ErrandResultInfo
   - completed: bool
   - result_cid: Vec<u8>
   - failed_count: u32
  operate_local_storage(ErrandResultInfo) -> store into local storage
end note

alice-[#blue]>alice:<OffChainWorker> update result of task
rnote right
update_errand_task_results(block_number) 
 results = get_from_local_storage
 range results {
   update_single_errand
   update_errand(description_cid,result_cid) {
       update Errand status and result
       remove from ProcessingErrands(description_cid)
   }
 }
end note

@enduml
```