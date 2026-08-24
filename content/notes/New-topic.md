+++
title = "Your New Note Title"
date = 2026-04-26
draft = true
+++
```toml
/* Hide the default theme header only on the Notes section */
.type-notes .pv4.pv6-l, 
.type-notes .tc-l.ph3.ph4-ns,
.type-notes header.white-70 {
    display: none !important;
}

/* Ensure the main content starts immediately below the nav */
.type-notes main {
    padding-top: 0 !important;
    margin-top: 0 !important;
}
/* Custom Inline Code Styling - April 2026 */
code {
  font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
  font-size: 0.85em;      
  background-color: #f6f8fa; 
  padding: 0.2em 0.4em;   
  border-radius: 3px;     
  color: #c7254e;         
  word-break: break-word;
}
```