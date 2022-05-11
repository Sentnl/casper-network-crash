## Bug Bounty for P0 Network security issue on a layer 1 blockchain protocol



### 1 We created a 5 node local testnet utilising: 
 
https://github.com/casper-network/casper-node/tree/master/utils/nctl 
Branch: release-1.4.3
 
### 2 Node and client Versions:
 
**Casper client:** 1.4.3
**Casper-node:** casper-node 1.4.3-9aa22ac6c-casper-mainnet 
 
### 3 Node setup was completed using :
 
```nctl-assets-setup```

### 4 Steps to crash network:


1.  Started casper-network testnet using ```nctl-start``
2.  Uploaded wasm contract with test1 account key using RPC of node-1

```./casper-client put-deploy --chain-name casper-net-1 --node-address http://localhost:11101/rpc --secret-key /home/charles/casper-node/utils/nctl/assets/net-1/users/user-1/secret_key.pem --session-path /home/charles/casper_crasher_2.wasm --payment-amount 10000000```

3. After consensus is reached (about 60 seconds) all nodes crash.
4. Error in log - panicked at ```'Own memory entry to exist in memory section', /home/charles/.cargo/registry/src/github.com-1ecc6299db9ec823/pwasm-utils-0.16.0/src/ext.rs:39:10```
 
### 5 Details on the WASM

There isn't really any source code to the wasm; it was found through fuzzing, which mutates binary wasm files. However, we can include all the necessary files to replicate the crash. 
 
**Tested with:** cargo 1.58.0-nightly (ad50d0d26 2021-11-17)
 
Files that are available upon request are: 
 
- **crasher.wasm** - This is the file which crashes the nodes
- **crasher.wat** - This is the text version of crasher.wasm
- **casper-poc/** - This is a Rust project which invokes your code with the crafted payload


### Configuration files:
 

Config.tml (typical node config)

```
[node]
 
[logging]
format = "json"
color = false
abbreviate_modules = false
 
[consensus]
secret_key_path = "../../keys/secret_key.pem"
 
[network]
public_address = "127.0.0.1:0"
bind_address = "0.0.0.0:22101"
known_addresses = [ "127.0.0.1:22101", "127.0.0.1:22102", "127.0.0.1:22103",]
gossip_interval = "30sec"
initial_gossip_delay = "5sec"
max_addr_pending_time = "1min"
max_outgoing_byte_rate_non_validators = 0
max_incoming_message_rate_non_validators = 0
 
[rpc_server]
address = "0.0.0.0:11101"
qps_limit = 100
 
[rest_server]
address = "0.0.0.0:14101"
qps_limit = 100
 
[event_stream_server]
address = "0.0.0.0:18101"
event_stream_buffer_length = 5000
max_concurrent_subscribers = 100
 
[storage]
path = "../../storage"
max_block_store_size = 19327352832
max_deploy_store_size = 12884901888
max_deploy_metadata_store_size = 12884901888
max_state_store_size = 10737418240
enable_mem_deduplication = false
mem_pool_prune_interval = 1024
 
[gossip]
infection_target = 3
saturation_limit_percent = 80
finished_entry_duration = "60sec"
gossip_request_timeout = "10sec"
get_remainder_timeout = "5sec"
 
[fetcher]
get_from_peer_timeout = "3sec"
 
[deploy_acceptor]
verify_accounts = true
 
[contract_runtime]
 
[linear_chain_sync]
sync_timeout = "1hr"
 
[block_proposer]
 
[consensus.highway]
pending_vertex_timeout = "1min"
request_state_interval = "20sec"
log_participation_interval = "1min"
log_synchronizer_interval = "5sec"
log_unit_sizes = false
max_execution_delay = 3
max_requests_for_vertex = 5
max_request_batch_size = 20
 
[network.estimator_weights]
consensus = 0
deploy_requests = 1
 
[consensus.highway.round_success_meter]
num_rounds_to_consider = 40
num_rounds_slowdown = 10
num_rounds_speedup = 32
acceleration_parameter = 40
```

Chainsepc.toml

```
[protocol]
version = "1.0.0"
hard_reset = false
activation_point = "2022-03-08T19:20:30.712205Z"
 
[network]
name = "casper-net-1"
maximum_net_message_size = 23068672
 
[core]
era_duration = "41seconds"
minimum_era_height = 10
validator_slots = 10
auction_delay = 3
locked_funds_period = "90days"
unbonding_delay = 14
round_seigniorage_rate = [ 15959, 6204824582392,]
max_associated_keys = 100
max_runtime_call_stack_height = 12
 
[highway]
finality_threshold_fraction = [ 1, 3,]
minimum_round_exponent = 12
maximum_round_exponent = 19
reduced_reward_multiplier = [ 1, 5,]
 
[deploys]
max_payment_cost = "0"
max_ttl = "1day"
max_dependencies = 10
max_block_size = 10485760
max_deploy_size = 1048576
block_max_deploy_count = 100
block_max_transfer_count = 1000
block_gas_limit = 10000000000000
payment_args_max_length = 1024
session_args_max_length = 1024
native_transfer_minimum_motes = 2500000000
 
[wasm]
max_memory = 64
max_stack_height = 188
 
[system_costs]
wasmless_transfer_cost = 100000000
 
[wasm.storage_costs]
gas_per_byte = 630000
 
[wasm.opcode_costs]
bit = 300
add = 210
mul = 240
div = 320
load = 2500
store = 4700
const = 110
local = 390
global = 390
control_flow = 440
integer_comparison = 250
conversion = 420
unreachable = 270
nop = 200
current_memory = 290
grow_memory = 240000
regular = 210
 
[system_costs.auction_costs]
get_era_validators = 10000
read_seigniorage_recipients = 10000
add_bid = 10000
withdraw_bid = 10000
delegate = 10000
undelegate = 10000
run_auction = 10000
slash = 10000
distribute = 10000
withdraw_delegator_reward = 10000
withdraw_validator_reward = 10000
read_era_id = 10000
activate_bid = 10000
 
[system_costs.mint_costs]
mint = 2500000000
reduce_total_supply = 10000
create = 2500000000
balance = 10000
transfer = 10000
read_base_round_reward = 10000
 
[system_costs.handle_payment_costs]
get_payment_purse = 10000
set_refund_purse = 10000
get_refund_purse = 10000
finalize_payment = 10000
 
[system_costs.standard_payment_costs]
pay = 10000
 
[wasm.host_function_costs.add]
cost = 5800
arguments = [ 0, 0, 0, 0,]
 
[wasm.host_function_costs.add_associated_key]
cost = 9000
arguments = [ 0, 0, 0,]
 
[wasm.host_function_costs.add_contract_version]
cost = 200
arguments = [ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.blake2b]
cost = 200
arguments = [ 0, 0, 0, 0,]
 
[wasm.host_function_costs.call_contract]
cost = 4500
arguments = [ 0, 0, 0, 0, 0, 420, 0,]
 
[wasm.host_function_costs.call_versioned_contract]
cost = 200
arguments = [ 0, 0, 0, 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.create_contract_package_at_hash]
cost = 200
arguments = [ 0, 0,]
 
[wasm.host_function_costs.create_contract_user_group]
cost = 200
arguments = [ 0, 0, 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.create_purse]
cost = 2500000000
arguments = [ 0, 0,]
 
[wasm.host_function_costs.disable_contract_version]
cost = 200
arguments = [ 0, 0, 0, 0,]
 
[wasm.host_function_costs.get_balance]
cost = 3800
arguments = [ 0, 0, 0,]
 
[wasm.host_function_costs.get_blocktime]
cost = 330
arguments = [ 0,]
 
[wasm.host_function_costs.get_caller]
cost = 380
arguments = [ 0,]
 
[wasm.host_function_costs.get_key]
cost = 2000
arguments = [ 0, 440, 0, 0, 0,]
 
[wasm.host_function_costs.get_main_purse]
cost = 1300
arguments = [ 0,]
 
[wasm.host_function_costs.get_named_arg]
cost = 200
arguments = [ 0, 0, 0, 0,]
 
[wasm.host_function_costs.get_named_arg_size]
cost = 200
arguments = [ 0, 0, 0,]
 
[wasm.host_function_costs.get_phase]
cost = 710
arguments = [ 0,]
 
[wasm.host_function_costs.get_system_contract]
cost = 1100
arguments = [ 0, 0, 0,]
 
[wasm.host_function_costs.has_key]
cost = 1500
arguments = [ 0, 840,]
 
[wasm.host_function_costs.is_valid_uref]
cost = 760
arguments = [ 0, 0,]
 
[wasm.host_function_costs.load_named_keys]
cost = 42000
arguments = [ 0, 0,]
 
[wasm.host_function_costs.new_uref]
cost = 17000
arguments = [ 0, 0, 590,]
 
[wasm.host_function_costs.print]
cost = 20000
arguments = [ 0, 4600,]
 
[wasm.host_function_costs.provision_contract_user_group_uref]
cost = 200
arguments = [ 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.put_key]
cost = 38000
arguments = [ 0, 1100, 0, 0,]
 
[wasm.host_function_costs.read_host_buffer]
cost = 3500
arguments = [ 0, 310, 0,]
 
[wasm.host_function_costs.read_value]
cost = 6000
arguments = [ 0, 0, 0,]
 
[wasm.host_function_costs.read_value_local]
cost = 5500
arguments = [ 0, 590, 0,]
 
[wasm.host_function_costs.remove_associated_key]
cost = 4200
arguments = [ 0, 0,]
 
[wasm.host_function_costs.remove_contract_user_group]
cost = 200
arguments = [ 0, 0, 0, 0,]
 
[wasm.host_function_costs.remove_contract_user_group_urefs]
cost = 200
arguments = [ 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.remove_key]
cost = 61000
arguments = [ 0, 3200,]
 
[wasm.host_function_costs.ret]
cost = 23000
arguments = [ 0, 420000,]
 
[wasm.host_function_costs.revert]
cost = 500
arguments = [ 0,]
 
[wasm.host_function_costs.set_action_threshold]
cost = 74000
arguments = [ 0, 0,]
 
[wasm.host_function_costs.transfer_from_purse_to_account]
cost = 2500000000
arguments = [ 0, 0, 0, 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.transfer_from_purse_to_purse]
cost = 82000
arguments = [ 0, 0, 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.transfer_to_account]
cost = 2500000000
arguments = [ 0, 0, 0, 0, 0, 0, 0,]
 
[wasm.host_function_costs.update_associated_key]
cost = 4200
arguments = [ 0, 0, 0,]
 
[wasm.host_function_costs.write]
cost = 14000
arguments = [ 0, 0, 0, 980,]
 
[wasm.host_function_costs.write_local]
cost = 9500
arguments = [ 0, 1800, 0, 520,]
```
 


