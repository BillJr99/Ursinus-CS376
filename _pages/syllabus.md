---
layout: syllabus
permalink: /
title: "CS376: Operating Systems"
excerpt: "CS376: Operating Systems"
    
info:
  course_number: CS376
  course_sections: 
  - section: "A"
  course_title: "Operating Systems"
  credit_hours: "4 Semester Hours"
  course_homepage: "https://www.billmongan.com/Ursinus-CS376-Spring2024/"
  class_notebook: https://ursinuscollege365-my.sharepoint.com/personal/wmongan_ursinus_edu/Documents/Class%20Notebooks/CS376%20Spring%202024
  ical: files/CS376.ics
  course_prerequisites: "CS274 Computer Architecture"
  course_start_date: "2024/01/15"
  course_end_date: "2024/05/01"
  course_description: "Fundamental concepts of operating systems. Sequential processes, concurrent processes, resource management, scheduling, synchronization, file systems, and computer security. Projects include writing of a program to simulate major components of an operating system. Pre- or co-requisite: CS-274. Offered in the spring of even years. Three hours per week. Four semester hours."
  welcome_message: "Welcome to CS376!"
  class_meets_days:
    isM: false
    isT: true
    isW: false
    isR: true
    isF: false 
    isS: false
    isU: false
  class_meets_locations:
  - section:
    - day: "T"
      starttime: "10:00 AM"
      endtime: "11:15 AM"
      place: "PFA 007"
    - day: "R"
      starttime: "10:00 AM"
      endtime: "11:15 AM"
      place: "PFA 007" 
  midtermexam: 
    - mdate: "TBD"
      mstarttime: "TBD"
      mendtime: "TBD"
      mroom: "TBD"       
  finalexam: 
    - fdate: "2023/12/14"
      fstarttime: "9:00 AM"
      fendtime: "12:00 PM"
      froom: "IDC 312"  
  flexible_submission_policy: "In the absence of <a href=\"#accommodations\">accommodations</a> arranged in advance with the instructor or college, all assignments are due at 10:59PM Eastern Time on the date(s) stated on the schedule.  Assignments will be accepted without prior permission following this time with a points deduction of 5% per day if submitted before 10:59 PM Eastern Time on the day submitted.<br><br>Students may request, in writing, up to three extensions during the semester, each lasting up to 7 days in duration.  This request should motivate the need, including the number of days requested, the reason for the request, and a day-by-day plan of one's time and energy; in addition, the request must include a report on progress to-date, including a copy of the deliverable in its current form, and documentation of at least one visit to the instructor's student hours or to the help room.  Each request will be granted only if such sufficient motivation is given, and only if the progress demonstrated merits a passing grade.  The request must be made at least 24 hours prior to the initial submission deadline.<br><br>Extra credit will not be awarded for assignments submitted under the flexible submission policy.  Students with accommodations will receive additional &quot;slack days&quot; as specified within the accommodations letter; however, some deliverables cannot be subject to accommodations due to the time-sensitive nature of the assignment (for example, group assignments, presentations, and course surveys).  Students who add the class late shall receive additional slack days equal to the number of days between the start of classes and the first date that approval is given or that class is attended (whichever occurs first).  Under no circumstances (including accommodations) can late work be accepted after the final class meeting, nor during final exams week, nor after the exam." 
  late_penalty_per_period: 5
  late_penalty_period: "day"
  banner: |
    <div style="width: 100%; display: table; border-collapse:separate; border-spacing:5px;">
    <div style="width: 100%; display: table-row;">
        <div style="display: table-cell; padding:5px; width:33%;">
            <a title="Maxtremus, CC0, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Round-robin_schedule_quantum_3.png"><img width="100%" alt="Round-robin schedule quantum 3" src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Round-robin_schedule_quantum_3.png/512px-Round-robin_schedule_quantum_3.png"></a>
        </div>
        <div style="display: table-cell; padding:5px; width:33%;">
            <a title="Huihermit, CC0, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:MINIX_3.2_Top_Command.png"><img width="100%" alt="MINIX 3.2 Top Command" src="https://upload.wikimedia.org/wikipedia/commons/2/2e/MINIX_3.2_Top_Command.png"></a>
        </div>
        <div style="display: table-cell; padding:5px; width:33%;">
            <a title="Traced by User:Stannered, original by en:User:Dysprosia, BSD &lt;http://opensource.org/licenses/bsd-license.php&gt;, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Virtual_address_space_and_physical_address_space_relationship.svg"><img width="100%" alt="Virtual address space and physical address space relationship" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Virtual_address_space_and_physical_address_space_relationship.svg/512px-Virtual_address_space_and_physical_address_space_relationship.svg.png"></a>
        </div>
    </div>
    </div>
    
instructors:
- name: William Mongan
  title: Professor
  email: wmongan@ursinus.edu
  phone: "610-409-3268"
  office: "Pfahler Hall 101L"
  webpage_url: "http://www.billmongan.com"
  picture: /images/profile.png
  officehourssignup: "https://cal.com/billmongan/10min"
  officehours:
  - day: "T"
    starttime: "11:30 AM"
    endtime: "2:30 PM"
    location: "Pfahler Hall 101L"  
  - day: "R"
    starttime: "11:30 AM"
    endtime: "2:30 PM"
    location: "Pfahler Hall 101L"         
    
textbooks:
- title: "Operating System Concepts"
  authors: "Abraham Silberschatz"
  edition: "10th Edition"
  isbn: "9781119320913"
  link: https://ursinus.bncollege.com/c/Operating-Systems-Concepts/p/MBS_2765951_dg
  isrequired: true 
  freelyavailable: false

objectives:
- objective: "To write thread-safe multithreaded programs"
- objective: "To utilize the programming facilities of modern operating systems for interprocess communication, including file system structures, sockets, shared memory, and I/O"
- objective: "To explain the structure and organization of a modern file system"
- objective: "To explain the principles of abstraction as they pertain to running processes on modern computing hardware"
- objective: "To utilize system calls to manage processes, memory, and file systems"
- objective: "To explain and manipulate the data structures and algorithms that underly modern operating systems"

goals:
- goal: "To use, develop and become familiar with programming constructs that interface with the operating system to provide functionality to the programmer and the user"
- goal: "To write portable systems-level applications"
- goal: "To coordinate threads using shared memory and distributed message-passing on a variety of platforms"
- goal: "To design and implement core operating system functionality"

grade_breakdown:
- category: "Programming Assignments"
  weight: "40%"
- category: "Labs"
  weight: "35%"
- category: "Midterm Exam"
  weight: "10%"
- category: "Final Project"
  weight: "15%"  

letter_grades:
- letter: "A+"
  range: "96.9-100"
- letter: "A"
  range: "93-96.89"
- letter: "A-"
  range: "89.5-92.99"
- letter: "B+"
  range: "87-89.49"
- letter: "B"
  range: "83-86.99"
- letter: "B-"
  range: "79.5-82.99"
- letter: "C+"
  range: "77-79.49"
- letter: "C"
  range: "73-76.99"
- letter: "C-"
  range: "69.5-72.99"
- letter: "D+"
  range: "67-69.49"
- letter: "D"
  range: "63-66.99"
- letter: "D-"
  range: "59.5-62.99"
- letter: "F"
  range: "0-59.49" 

schedule:
  - week: "0"
    date: "1"
    title: "Course Overview"
    link: "../Ursinus-CS376-Overview"
    deliverables:
      - dtitle: "Homework Assignment: Warmup Handed Out"
        dlink: "Assignments/Written/Warmup"
        points: 15
        submission_types: "onpaper"
  - week: "1"
    date: "0"
    title: "Course Overview"
    deliverables:
      - dtitle: "Homework Assignment: Warmup Due"
        dlink: "Assignments/Written/Warmup"
        points: 15
        submission_types: "onpaper"
        
university:
  semester: "Spring"
  academicyear: "2023-24"
  fall:
  - kname: "Add Deadline"
    kdate: "2023/09/8"
    kdisplay: true
  - kname: "Mid Semester Grades Posted"
    kdate: "2023/10/13"
    kdisplay: false    
  - kname: "Drop with a W Deadline"
    kdate: "2023/10/25"
    kdisplay: true  
  - kname: "Reading Day"
    kdate: "2023/12/9"
    kdisplay: true     
  - kname: "Finals Week Begins"
    kdate: "2023/12/11"
    kdisplay: false
  - kname: "Finals Week Ends"
    kdate: "2023/12/16"
    kdisplay: false
  spring:
  - kname: "Add Deadline"
    kdate: "2024/01/30"
    kdisplay: true
  - kname: "Mid Semester Grades Posted"
    kdate: "2024/03/1"
    kdisplay: false    
  - kname: "Drop with a W Deadline"
    kdate: "2024/03/20"
    kdisplay: true    
  - kname: "Reading Day"
    kdate: "2024/05/2"
    kdisplay: false    
  - kname: "Finals Week Begins"
    kdate: "2024/05/03"
    kdisplay: false
  - kname: "Finals Week Ends"
    kdate: "2024/05/09"
    kdisplay: false       
  - kname: "Baccalaureate"
    kdate: "2024/05/10"
    kdisplay: false
  - kname: "Commencement"
    kdate: "2024/05/11"
    kdisplay: false 
  fallholidays:
  - date: "2023/10/14"
  - date: "2023/10/15"
  - date: "2023/10/16"
  - date: "2023/10/17"
  - date: "2023/11/22"
  - date: "2023/11/23"
  - date: "2023/11/24"
  - date: "2023/11/25"
  - date: "2023/11/26"
  springholidays:
  - date: "2024/01/15"  
  - date: "2024/03/4"  
  - date: "2024/03/5"  
  - date: "2024/03/6"  
  - date: "2024/03/7"  
  - date: "2024/03/8" 
  
---

Welcome to CS376: Operating Systems!
