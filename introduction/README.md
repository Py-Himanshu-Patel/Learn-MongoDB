# Introduction

- The primary reason for moving away from the relational model is to make scaling out easier.
- Scaling a database comes down to the choice between scaling up (getting a bigger machine) or scaling out (partitioning data across more machines).
- MongoDB automatically takes care of balancing data and load across a cluster, redistributing documents automatically and routing user requests to the correct machines. This allows developers to focus on programming the application, not scaling it. When a cluster need more capacity, new machines can be added and MongoDB will figure out how the existing data should be spread to them.
