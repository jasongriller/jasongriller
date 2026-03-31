### Hi there 👋

<!--
**jasongriller/jasongriller** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->

● Good question. Let me trace the full chain:                                                                                                                                
                                               
  Today's flow (disconnected):                                                                                                                                                 User opens our app                                                                                                                                                         
    → Picks "Decompose Feature"                                                                                                                                              
    → Types: "Build a new candidate search feature"
    → Gets back 5 User Stories + 2 Technical Enablers
    → ...now what? They copy-paste into ADO manually. No link. No team. Nothing.

  The gap: Our app doesn't know WHO this person is in ADO terms. We know their Cognito login, but we don't know:
  - What ADO team are they on?
  - What Area Path do their work items go under?
  - What iteration/sprint are they in?
  - What parent Feature in ADO should these stories link to?

  The connection we'd build:

  Step 1: WHO are you?
    User picks their team from dropdown (fetched live from ADO)
    "Backlog to the Future" → we now know Area Path = AIE\Backlog to the Future

  Step 2: WHAT are you decomposing?
    Option A: User pastes an ADO Work Item ID (e.g., "Feature 12345")
      → We fetch that Feature from ADO
      → We know its title, description, area path, iteration
      → Generated stories become CHILDREN of Feature 12345

    Option B: User just types a description (no ADO link)
      → Stories get created under the team's area path
      → No parent link, but still on the right board

  Step 3: GENERATE (existing — no change)
    → 5 User Stories + 2 Technical Enablers come back

  Step 4: PUSH TO ADO (new)
    User clicks "Create in ADO"
    → For each generated story, we call ADO API:
       - System.Title = generated title
       - System.Description = generated description + acceptance criteria
       - System.AreaPath = "AIE\Backlog to the Future" (from Step 1)
       - System.IterationPath = current sprint (from team settings)
       - Parent link = Feature 12345 (from Step 2, if provided)
    → Stories appear on that team's board, linked to the parent Feature

  The key insight: The connection is Area Path + Parent Work Item ID. Area Path puts it on the right team's board. Parent ID links it to the Feature being decomposed. Both  
  come from ADO's API — we just need the user to tell us their team and (optionally) the Feature ID.
                                                                                                                                                                             ● User answered Claude's questions:                                                                                                                                          
  ⎿  · Does this connection chain make sense? Is the 'pick your team + optionally paste a Feature ID' approach what you'd want, or do you see it working differently? → Makes      sense                                                                                                                                                                     
● Good. So the two pieces that make the connection are:                                                                                                                      

  1. Team dropdown → gives us the Area Path → stories land on the right board                                                                                                
  2. Feature ID (optional) → gives us the parent link → stories are children of that Feature                                                                                                                                                                                                                                                              
  Everything else (title, description, acceptance criteria, story points) we already generate. We just need those two inputs from the user to close the loop to ADO.                                                                                                                                                                                        Want me to update the technical plan with this mapping flow, or are you still in "talk to boss" mode?                                                                                                                                                                                                                                                   ✻ Baked for 31s                                                                                                                                                              
                                                                                                                                                                             
❯ could u shorten it  a bit maybe ping to boss english kinda write                                                                                                                                                                                                                                                                                        
● ---                                                                                                                                                                          ADO Integration — What I'm Thinking                                                                                                                                        
                                                                                                                                                                               We've been getting requests from teams to connect our app directly to ADO. Right now everything we generate (stories, sprint notes, etc.) has to be manually copied into   
  ADO. I want to close that gap.                                                                                                                                             
     
  Here's the concept: when someone generates a Decompose or User Story in our tool, we can map it back to the right team in ADO automatically.

  How: ADO uses Area Paths to assign work items to teams. Our team names already match ADO's exactly. So we query ADO, get the mapping, and when a user picks their team and 
  hits generate — we already know where those stories belong. From there they can push them straight into ADO as real work items, linked to the parent Feature, on the right 
  team's board.

  We already have 3 services pulling data FROM ADO (wiki sync, inventory digest, repo digest). Same auth, same infrastructure. We'd just be adding the ability to write back.

  Want to start with a small proof of concept — just the team mapping + pushing one generated story into ADO. Low effort, high signal on whether this works.
