<!--
=============================================================================
SYNC IMPACT REPORT - Constitution Update v1.0.0
=============================================================================
VERSION CHANGE: New (initialized 2025-12-05)
CHANGE TYPE: MINOR (comprehensive constitution for new program)

PRINCIPLES (6 total):
1. Embodied Excellence - Physical systems integration and hands-on learning
2. Collaborative Intelligence & Code Quality - Peer review, reproducibility
3. Simulation-First Safety & Responsibility - Safety checkpoints, ethical bias testing
4. Open Standards & Reproducibility - Industry tools, documented experiments
5. Ethical Design & Environmental Stewardship - Safety, transparency, resource management
6. Continuous Learning & Constitutional Evolution - Quarterly amendments, evidence-driven

SECTIONS ADDED:
- II. Learning Objectives & Competencies
- III. Academic Integrity & Code Standards
- IV. Hardware Access & Lab Governance
- V. Simulation & Deployment Workflow
- VI. Assessment & Capstone Expectations
- VII. Intellectual Property & Publication
- VIII. Collaboration & Conflict Resolution
- IX. Governance (Amendment Process & Compliance)

TEMPLATES REQUIRING UPDATES:
- ✅ plan-template.md: "Constitution Check" already present, aligns with principles
- ✅ spec-template.md: Already captures independent testability, aligns with P2 (Code Quality)
- ✅ tasks-template.md: Task organization by user story aligns with P2 & P4
- ⚠ phr-template.md: No updates needed; template agnostic to principles

FOLLOW-UP TODOs:
- None deferred; all placeholders filled

=============================================================================
-->

# Physical AI & Humanoid Robotics: Program Constitution

**Preamble**: We, the faculty and students of the Physical AI & Humanoid Robotics capstone program, establish this constitution to guide our shared mission of bridging the gap between digital intelligence and embodied systems. We recognize that the future of AI extends beyond screens and into the physical world—a transition that demands rigorous technical discipline, ethical responsibility, and collaborative excellence. This constitution serves as our social contract: a binding agreement on principles, responsibilities, and expectations.

---

## I. Core Principles

### Principle 1: Embodied Excellence
Every student in this program must achieve hands-on understanding of physical systems. Simulation without deployment, and theory without practice, are incomplete. All work follows a mandatory pipeline: Design in simulation → Algorithm development → Data generation → Deployment readiness → Physical validation. Students must demonstrate mastery across the full robotics stack: sensors → fusion → cognition → planning → control → actuation → feedback.

### Principle 2: Collaborative Intelligence & Code Quality
This program is built on the principle that complex systems require diverse expertise and rigorous software engineering. All code must be original (not copy-pasted), properly attributed, version-controlled with clear commit messages, and subject to mandatory peer review before merging to main branches. Code reviews are learning opportunities—not blame sessions. Reproducibility is non-negotiable: every experiment must document hardware configuration, software versions, random seeds, and simulation parameters. The "Blame Game" is forbidden; git history exists for improvement, not punishment.

### Principle 3: Simulation-First Safety & Responsibility
All robots designed in this program must prioritize safety, transparency, and beneficial outcomes. Before any real robot receives a command from student-written code, two faculty members must sign off on a safety review. A simulated dry run must complete without crashes or unexpected behaviors. Emergency stop buttons must be within reach and tested. All students must be briefed on failure modes. We acknowledge that vision models inherit bias from training data; all perception pipelines must be tested for performance across demographic groups. Documented bias is acceptable; undiscovered bias is not.

### Principle 4: Open Standards & Reproducibility
We commit to industry-standard tools (ROS 2, NVIDIA Isaac Sim, Gazebo) and reproducible practices, ensuring our work contributes to the broader robotics ecosystem. All work must be reproducible: hardware specs (GPU, VRAM, CPU, RAM), software versions (ROS 2, Python, dependencies), random seeds, and simulation parameters are documented. Irreproducible work (e.g., "it worked once but I can't replicate it") cannot be submitted for assessment. Documentation must include clear separation of concerns, minimal technical debt, and interpretable decision-making (especially when using LLMs).

### Principle 5: Ethical Design & Environmental Stewardship
Robots operating in human spaces carry ethical weight. All designs must include failsafes, bounded action spaces, and transparent decision-making. Robots must not be used for surveillance, manipulation, or deception without explicit informed consent. Computational resources are finite; students must minimize unnecessary training runs and GPU cycles. Simulation is preferred over repeated real-world trials. Students acknowledge the environmental impact of high-performance compute and act as responsible stewards.

### Principle 6: Continuous Learning & Constitutional Evolution
This program is not static. At the end of each quarter, students and faculty convene a Constitutional Convention to reflect on what worked, propose amendments, and vote on changes (simple majority). Amendments may be proposed by any participant with 2 weeks' notice, rationale, and impact analysis. Principles must remain declarative, testable, and free of vague language. All principles are measured by demonstrated mastery and evidence-driven outcomes.

---

## II. Learning Objectives & Competencies

Upon completion of this program, students will demonstrate mastery in:

- **Physical AI Fundamentals**: Understanding embodied intelligence, sensor integration, and the translation of digital algorithms into physical action.
- **ROS 2 Architecture**: Designing modular, distributed systems using nodes, topics, services, and actions.
- **Simulation & Digital Twins**: Creating photorealistic simulations that enable safe training before real-world deployment.
- **Perception & Navigation**: Implementing VSLAM, computer vision, and path planning for autonomous operation.
- **Language-Guided Reasoning**: Integrating large language models to convert natural language commands into executable robot behaviors.
- **Sim-to-Real Transfer**: Bridging the gap between simulation and physical deployment with robust, production-ready code.

---

## III. Academic Integrity & Code Standards

### Original Work Requirement
Students must write original Python code for ROS 2 packages, perception pipelines, and planning modules. Permitted: using official library APIs (rclpy, opencv, nav2), following tutorials, and integrating pre-trained models (Whisper, LLaMA). Prohibited: copy-pasting solutions from peers, previous cohorts, or external sources without attribution. All code must include citations to external resources (papers, open-source projects, official examples).

### Version Control & Collaboration
All project code lives in version control (Git) with clear commit messages. Code reviews are mandatory before merging to main branches; at least one peer reviewer must approve. Merge conflicts are opportunities to learn, not occasions for blame. All team members are accountable for the final submission.

### Reproducibility Standards
Every experiment must be reproducible. Students document: hardware configuration (GPU model, VRAM, CPU, RAM), software environment (ROS 2 version, Python version, dependency versions in requirements.txt), random seeds (for training stability), and simulation parameters (friction, gravity, sensor noise).

---

## IV. Hardware Access & Lab Governance

### Resource Allocation
The program operates under a resource-constrained reality. High-performance compute, sensors, and robots are shared infrastructure. Access is governed by:
- First-Come, First-Served Reservation System: Students book lab resources 48 hours in advance.
- Equitable Distribution: No single team monopolizes GPU time, sensors, or robots during the quarter.
- Peak vs. Off-Peak Usage: Students are encouraged to run long training jobs during off-peak hours (nights/weekends).

### Hardware Responsibility
Students working with lab hardware agree to careful handling (robots and sensors are expensive and irreplaceable mid-quarter), proper documentation of issues within 24 hours, clean shutdown of all processes, and calibration/maintenance ownership with deviations documented.

### Cloud vs. Local Execution
Simulation Training: Students may use AWS cloud instances (within institutional budget: $205/quarter per cohort) for long-running physics simulations. Model Deployment: Final model weights must be downloaded and flashed to local Jetson kits. Real-time robot control from the cloud is prohibited due to latency and safety risks.

---

## V. Simulation & Deployment Workflow

### Mandatory Pipeline
All work follows these stages:
1. **Design Phase** (Digital Twin in Gazebo/Isaac Sim): Build URDF models, test physics, validate sensor data streams.
2. **Algorithm Development** (Python in ROS 2): Write perception, planning, and control algorithms; test in simulation.
3. **Data Generation** (Synthetic Data from Isaac Sim): Generate labeled datasets for training vision and manipulation models.
4. **Deployment Readiness** (Model Quantization & Edge Testing): Optimize models for Jetson Orin Nano; test offline.
5. **Physical Deployment** (Real Robot Deployment): Only after all stages pass review do students deploy to real hardware.

### Safety Checkpoints
Before any real robot receives a command from student-written code: Two faculty members must sign off on safety review. Simulated dry run must complete without crashes. Emergency stop buttons must be within reach and tested. All students in the room must be briefed on failure modes.

---

## VI. Assessment & Capstone Expectations

### Continuous Evaluation
- **ROS 2 Package Development** (Weeks 3–5): Submit a working ROS 2 package with nodes, services, and parameter management. Evaluated on code quality and architectural decisions.
- **Gazebo Simulation** (Weeks 6–7): Complete URDF model with sensor simulation and basic physics validation.
- **Isaac Perception Pipeline** (Weeks 8–10): Trained perception model (SLAM or object detection) deployed to Jetson kit.
- **Capstone Project** (Weeks 11–13): Simulated (and optionally real) humanoid robot that accepts voice commands, plans paths, navigates obstacles, identifies objects, and manipulates them. Must work reliably in at least 3 different scenarios.

### Capstone Rubric
- **Technical Soundness (40%)**: Does the system work as designed? Are algorithms robust to sensor noise and environmental variation?
- **Code Quality (20%)**: Is code modular, well-documented, and maintainable? Clear separation of concerns and minimal technical debt?
- **Innovation & Insight (20%)**: Does the project demonstrate understanding beyond the curriculum? Novel solutions to identified problems?
- **Reproducibility & Safety (20%)**: Can another student run this code without friction? Safety practices evident throughout?

---

## VII. Intellectual Property & Publication

### Ownership
- **Student Work**: Code and algorithms developed by students are their intellectual property. Students may publish, open-source, or commercialize their work.
- **Curriculum**: Course materials, URDF models, and simulation environments may be shared with future cohorts under a Creative Commons license.

### Attribution
Students publishing research based on this capstone must cite the course and advisors. Shared contributions to the lab's codebase are tracked and publicly credited in documentation.

---

## VIII. Collaboration & Conflict Resolution

### Team Dynamics
Teams of 2–4 students collaborate on projects. Clear roles (Architecture Lead, Perception Specialist, Deployment Engineer, etc.) are assigned. Weekly standups (15 minutes) synchronize progress and surface blockers. All team members are accountable for the final submission.

### Dispute Resolution
1. **Peer Resolution** (72 hours): Teams attempt to resolve internally, documented in writing.
2. **Faculty Mediation** (1 week): If unresolved, faculty facilitates a structured discussion.
3. **Administrative Review** (2 weeks): If mediation fails, the department reviews and arbitrates.

---

## IX. Governance

This constitution is the authoritative guide for all program activities. All policies, rubrics, and procedures must align with these principles. Any conflicts default to the constitution.

### Amendment Process
Amendments may be proposed by any student or faculty member with 2 weeks' notice before end-of-quarter Constitutional Convention. Proposals must include rationale and impact analysis. Changes are approved by simple majority vote among active participants.

### Compliance & Oversight
- All project submissions must verify compliance with principles (especially Reproducibility, Safety, and Ethical Design).
- Faculty conduct quarterly reviews to assess alignment with constitution principles.
- This constitution is reviewed annually and refined based on participant feedback.

---

**Version**: 1.0.0 | **Ratified**: 2025-12-05 | **Last Amended**: 2025-12-05
