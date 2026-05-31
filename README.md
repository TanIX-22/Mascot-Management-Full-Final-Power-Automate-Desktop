# Mascot-Management-Full-Final-Power-Automate-Desktop
A lean UiPath bot for ESIC portal registration. Operates sequentially in a single browser session to eliminate heavy retry overhead. Features Smart App State validation to instantly flag existing PANs, logging duplicates as "Already Registered" in Excel. Downloads and routes generated ID cards directly into target employee folders cleanly.
## Visual Workflow Architecture

```mermaid
graph TD
    A[Start: Read Master Excel Data] --> B[Sanitize Data & Format Variables]
    B --> C[For Each Row / Employee]
    
    subgraph Loop Iteration [PAD & VBScript Execution Loop]
        C --> D[Evaluate Row Integrity & Drop Empties]
        D --> E{Conditional Logic:<br>Form I Eligible?}
        
        E -- Yes --> F[Flag Form I Template + F&F Template]
        E -- No --> G[Flag F&F Template Only]
        
        F --> H[Invoke Run VBScript Action]
        G --> H
        
        subgraph VBScript Background Processing
            H --> I[Launch Headless Word instance<br>Visible = False]
            I --> J[Execute String-Safe Find & Replace]
            J --> K[Save Document with Unique Naming]
            K --> L[Close Word Instance & Free RAM]
        end
        
        L --> M{Check VBScript Run Status}
        
        M -- Failure / Error --> N[Log 'Failed' + Error Message to Excel]
        M -- Success --> O[Log 'Success' + Output Path to Excel]
    end
    
    N --> P{Has Next Row?}
    O --> P
    P -- Yes --> C
    P -- No --> Q[End: Save & Close Master Excel Tracker]
