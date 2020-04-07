# Arrow Compression

## Overview
>What motivates this to be implemented? What will this component achieve? 

## Scope
>Which parts of the system will this feature rely on or modify? Write down specifics so people involved can review the design doc

## Glossary (Optional)
As of now, we have not added any unintuitive concepts or names of components.

## Architectural Design
>Explain the input and output of the component, describe interactions and breakdown the smaller components if any. Include diagrams if appropriate.

## Design Rationale
>Explain the goals of this design and how the design achieves these goals. Present alternatives considered and document why they are not chosen.

## Testing Plan
First, we would like to establish the baseline performance for OLAP queries by running a standard OLAP workload like TPC-H on terrier. With this information in hand, we can measure the speedup that our implementation provides for OLAP queries.

We would then enable the compaction process, verify the correctness of the arrow format blocks and then run the same OLAP queries on these compressed blocks. At this stage, the reader will still be materializing the tuples to be read and so we do not expect any performance improvements here. 

Our final testing step would be to run the same OLAP queries directly on the compressed data without materializing tuples to be read. We expect a significant performance improvement here because there is no overhead of materialising tuples and also that tuples within a block are present in contiguous memory locations. 

If we get to the 125% project goal (designing policy for marking a block for conversion), we can also run rigorous tests with different policies on hybrid workloads. This way, we can determine the best policy for marking a block for compaction.

## Trade-offs and Potential Problems
>Write down any conscious trade-off you made that can be problematic in the future, or any problems discovered during the design process that remain unaddressed (technical debts).

## Future Work
Currently we are working on our 75% goal which is to enable compression of blocks through the execution layer (updates tables as well as indexes). We are yet to start working on the execution layer but we think we will have to write TPL code which will be called as a builtin from the `block_compactor` code. 

For our 100% goal, we will be running OLAP queries directly on top of the compressed data and so, we will have to add functionality for reading FROZEN blocks directly rather than materializing them. 

For the 125% goal we have to experiment with the different policies for starting the cooling process. This would involve changes to the policy in the garbage collector code.
