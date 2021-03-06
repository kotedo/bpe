Business Process Engine
=======================

Overview
--------

BPE is Synrc Cloud Stack Erlang Application that brings Erlang for Enterprises.
It provides infrastructure for Workflow Definitions, Process Orchestration,
Rule Based Production Systems and Distributed Storage.

Workflow Engine
===============

Workflow Engine -- is an Erlang/OTP application which handles process definitions,
process instances, and provides very clean API for Workplaces.

Process Schema
--------------

Before using Process Engine you need to define the set of business process
workflows of your enterprise. This could be done via Erlang terms or some DSL
that lately should be converted to Erlang terms. Internally BPE uses Erlang terms
workflow definition:

```erlang
bpe:start(
   #process{name="Order11",
       flows=[ #sequenceFlow{source="begin",target="end2"},
               #sequenceFlow{source="end2",target="end"}],
       tasks=[ #userTask{name="begin"},
               #userTask{name="end2"},
               #endEvent{name="end"}],
       task="begin",beginEvent="begin",endEvent="end"},[]).

```

The workflow definition uses following persistent workflow model which is stored in KVS:

```erlang
-record(task,         { name, id, roles, module }).
-record(userTask,     { name, id, roles, module }).
-record(serviceTask,  { name, id, roles, module }).
-record(messageEvent, { name, id, payload }).
-record(beginEvent ,   { name, id }).
-record(endEvent,      { name, id }).
-record(sequenceFlow, { name, id, source, target }).
-record(history,      { ?ITERATOR(feed,true), name, task }).
-record(process,      { ?ITERATOR(feed,true), name,
                        roles=[], tasks=[], events=[], history=[], flows=[],
                        rules, docs=[],
                        task,
                        beginEvent, endEvent }).
```

Full set of BPMN 2.0 fields could be obtained at [http://www.omg.org/spec/BPMN/2.0](http://www.omg.org/spec/BPMN/2.0) page 3-7.

Sample Session
--------------

```erlang
(bpe@127.0.0.1)1> kvs:join().
ok
(bpe@127.0.0.1)1> rr(bpe).
[beginEvent,container,endEvent,history,id_seq,iterator,
 messageEvent,process,sequenceFlow,serviceTask,task,userTask]
(bpe@127.0.0.1)2> bpe:start(#process{name="Order11",
         flows=[#sequenceFlow{source="begin",target="end2"},
                #sequenceFlow{source="end2",target="end"}],
         tasks=[#userTask{name="begin"},
                #userTask{name="end2"},
                #endEvent{name="end"}],
         task="begin",beginEvent="begin",endEvent="end"},[]).
bpe_proc:Process 39 spawned <0.12399.0>
{ok,<0.12399.0>}
(bpe@127.0.0.1)3> gen_server:call(pid(0,12399,0),{complete}).
(bpe@127.0.0.1)4> gen_server:call(pid(0,12399,0),{complete}).
(bpe@127.0.0.1)5> gen_server:call(pid(0,12399,0),{complete}).
(bpe@127.0.0.1)5> bpe:history(39).
[#history{id = 28,version = undefined,container = feed,
          feed_id = {history,39},
          prev = 27,next = undefined,feeds = [],guard = true,
          etc = undefined,name = "Order11",
          task = {task,"end"}},
 #history{id = 27,version = undefined,container = feed,
          feed_id = {history,39},
          prev = 26,next = 28,feeds = [],guard = true,etc = undefined,
          name = "Order11",
          task = {task,"end2"}},
 #history{id = 26,version = undefined,container = feed,
          feed_id = {history,39},
          prev = undefined,next = 27,feeds = [],guard = true,
          etc = undefined,name = "Order11",
          task = {task,"begin"}}]
```

Process Instances
-----------------

Instantiation of process means creating persistent context of document flow.

```erlang
load(ProcName)
start(Proc,Docs)
amend(Proc,Docs)
complete(Proc)
history(ProcId)
task(Name,Proc)
doc(Name,Proc)
events(Proc)
tasks(Proc)
```

Using 'tasks' API you can fetch current documents attached to the given
process at particular stage. Using 'amend' API you can upload or
change document at current stage. 'push' API moves current
stage documents further by workflow.

Let us see how we could create initial 'Wire Transfer' transaction:

```erlang
bpe:load("WireTransfer"),
Id = bpe:start('WireTransfer',[]),
[] = bpe:tasks(Id), % current set is empty
Tasks = [ #'WireTransferReq'{
            beneficiary = #agent{ bank="SBININBB380",
                                  name="Namdak Tonpa",
                                  account="305820317"},
            subsidiary = #agent { bank="BKTRUS33",
                                  name="Maxim Sokhatsky",
                                  account="804250223"}}],

bpe:amend(Id,Tasks),
bpe:push(Id),
```

Dialyzer
--------

For check API with dialyzer build with `rebar compile` and run `make dialyze`.

Credits
-------

* Maxim Sokhatsky

OM A HUM
