# **Title:**

**Comparative Analysis of Current, Cached, and Archived Web Pages for OSINT and Cybersecurity Investigations**



## **Objective**

To analyze changes made to a webpage by comparing its current version, search engine–cached version, and archived version. This supports cybersecurity and OSINT investigations by identifying deleted or modified content.


## **Background**

Cached versions of web pages—stored by search engines—can reveal content no longer available on the live site. Similarly, archived versions captured by services like the Wayback Machine provide a historical record of the page.
This analysis helps:

* Identify attempts by threat actors to hide or remove content.
* Uncover changes in narratives or publication dates.
* Assist in digital forensics and OSINT research.



## **Task Breakdown**

### **1. Goal**

Manually compare and analyze the differences among:

* The current version of a webpage
* Its most recent cached version (via a search engine)
* Its archived version (via archival services)



### **2. Lab Setup**

Set up the tools and sources needed for the exercise:

* **Search Engine with Cache Access:**
  Example: Yandex, Baidu, Google (in some regions)

* **Web Archiving Tools:**

  * [Wayback Machine](https://archive.org/web/)
  * [Archive.today](https://archive.today/)
  * [SearXNG](https://docs.searxng.org/)

---

### **3. Web Page Selection and Viewing**

#### 3.1 News Organization Webpage

* Select a live webpage (e.g., a recent or controversial article)
* View and save:

  * Current live version
  * Cached version from a search engine
  * Archived version from an archive service

#### 3.2 Cybersecurity Blog Article

* Select an article published over a year ago
* View and save:

  * Cached version via a search engine
  * Archived version(s) from archive tools

---

### **4. Analysis of Differences**

Compare all versions using the following **guiding questions**:

#### 4.1 Key Questions

* Have any **images or videos** been removed or replaced?
* Are there **broken links or redirects** in the current version?
* Do **comments or timestamps** suggest modifications?
* What can be inferred from **differences** in content, layout, or metadata?
* Were **names, titles, or any sensitive text** altered or omitted?

---

### **5. Report Writing**

#### 5.1 Report Composition

Prepare a professional PDF report that includes:

* **Cover Page**
* **Table of Contents**
* **Revision History**
* **Executive Summary**
* **Detailed Analysis**:

  * News organization webpage
  * Cybersecurity blog article
* **Conclusion**:

  * Summary of your findings
  * Importance to OSINT or cybersecurity goals
* **Appendix**:

  * Screenshots of all webpage versions
  * Captions and URLs for each screenshot

#### 5.2 Formatting Guidelines

* Use **consistent headings and fonts**
* Align **images and captions at the center**
* Ensure clear separation of sections and subsections
* Use bullet points or tables for easy comparison if needed
