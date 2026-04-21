# PES-VCS: A Lightweight Version Control System

**Student Name:** Devika N  
**SRN:** PES1UG24AM080  
**Course:** Operating Systems


---

## 1. Project Overview
This project is a functional, Git-inspired Version Control System (VCS) implemented in C. It features a sharded object store for content-addressed storage, a staging area for managing changes, and a commit system to track repository history through linked objects.

---

## 2. Phase Screenshots

### Phase 1: Object Store
* **1A: Test Objects Output**
  
  <img width="940" height="214" alt="image" src="https://github.com/user-attachments/assets/e3c9fd83-59fb-49a1-8d71-c9fcb7972e82" />
  
    *Output showing passing tests for blob storage, deduplication, and integrity.* 

* **1B: Sharded Directory Structure**

<img width="940" height="123" alt="image" src="https://github.com/user-attachments/assets/5f651a8b-136a-47fd-a6a1-a1fc67f0ef47" />
    
    Verification of the .pes/objects directory structure showing two-character sharding. 

### Phase 2: Tree Construction
* **2A: Test Tree Output**

  <img width="940" height="206" alt="image" src="https://github.com/user-attachments/assets/a6fdfc8f-9d01-469c-847a-2e8b37228edf" />

    *Confirmation of successful tree serialization and deterministic hashing.* 

* **2B: Raw Tree Hex Dump**

  <img width="940" height="123" alt="image" src="https://github.com/user-attachments/assets/ac9d3d3c-2fc6-478c-9167-6d8bc3a48b27" />
  
    *Binary representation of a tree object showing file modes, hashes, and names.* 

### Phase 3: Staging Area (Index)
* **3A: Add and Status Sequence**

  <img width="940" height="206" alt="image" src="https://github.com/user-attachments/assets/707e41f7-ded2-4936-b463-1c90768a1382" />

    *Workflow demonstrating repository initialization and staging files for commit.*

* **3B: Index File Content**

  <img width="940" height="172" alt="image" src="https://github.com/user-attachments/assets/478a32bf-a164-4adc-b214-8d53f5313f2b" />
  
    *The internal state of the .pes/index file in text format.* 

### Phase 4: Commits and History
* **4A: Commit Log**

  <img width="940" height="125" alt="image" src="https://github.com/user-attachments/assets/f600b659-c0b8-4732-a666-f699790775d9" />

    *The output of `./pes log` showing the commit history, authors, and timestamps.*

* **4B: Object Growth**

  <img width="940" height="517" alt="image" src="https://github.com/user-attachments/assets/efe982e9-b91c-455f-b5bb-782a17d62ad6" />

    *A list of all objects generated in the .pes directory after several operations.* 

* **4C: Branch References**

  <img width="940" height="70" alt="image" src="https://github.com/user-attachments/assets/ed95ca68-b7f8-4c61-af15-782ac66ea181" />
 
    *The current hash stored in the main branch reference file.* 



### FINAL - make test-integration

<img width="940" height="435" alt="image" src="https://github.com/user-attachments/assets/4b7a2a86-b1e2-4389-82c6-afed82271852" />

<img width="940" height="424" alt="image" src="https://github.com/user-attachments/assets/a7d67356-598e-4650-9453-0377b007f5cc" />

<img width="940" height="479" alt="image" src="https://github.com/user-attachments/assets/6f5f202c-0524-4688-81d0-d71621ac4de3" />

<img width="940" height="419" alt="image" src="https://github.com/user-attachments/assets/76c93563-7ca6-4c9e-a0ec-48d8c125423a" />



---

## 3. Analysis Questions

### Section 5: Branching and Checkout

**Q5.1: Implementing `pes checkout <branch>`** To implement checkout, the `.pes/HEAD` file must be updated to point to the new branch reference (e.g., `ref: refs/heads/branch-name`). The working directory must then be synchronized with the target branch's tree by deleting files not present in the target and overwriting existing ones with the correct versions from the object store. This is complex because it must be done safely—if there are uncommitted changes that would be overwritten, the operation must be aborted to prevent data loss.

**Q5.2: Detecting "Dirty Working Directory" Conflicts** A "dirty" conflict is detected by comparing the working directory, the index, and the target tree. First, check if a file is modified by comparing its current `stat` data against the index. If it differs, the file is dirty. Second, compare the hash of that file in the index with its hash in the target branch's tree. If the file is dirty and the hashes differ, a conflict exists, and the checkout should be refused.

**Q5.3: Detached HEAD State** In a detached HEAD state, `HEAD` points directly to a commit hash instead of a branch name. Commits made in this state are successful, but they are not associated with any branch. If a user switches away to another branch, these commits become "orphaned" because no named reference points to them. They can be recovered by finding the commit hash in the log or object store and manually creating a new branch at that location.

### Section 6: Garbage Collection (GC)

**Q6.1: Reachability Algorithm** A Mark-and-Sweep algorithm is used for GC. The "Mark" phase starts at all reachable roots (branches, tags, and HEAD) and recursively traverses the graph of commits, trees, and blobs, adding their hashes to a "reachable" set (ideally a Hash Set for O(1) lookups). The "Sweep" phase then iterates through all files in `.pes/objects` and deletes any file whose hash is not in the reachable set. For 100,000 commits, the algorithm would visit at least 100,000 objects plus the unique trees and blobs associated with them.

**Q6.2: GC Race Conditions** Running GC concurrently with a commit is dangerous because a race condition can occur: a `commit` process might write a new blob to the store that is not yet linked to a tree or HEAD. If GC runs at that exact moment, it will see the blob as "unreachable" and delete it. When the commit process finally tries to reference that blob, the repository becomes corrupted. Git avoids this by using a grace period (pruning only objects older than a certain date).

---
