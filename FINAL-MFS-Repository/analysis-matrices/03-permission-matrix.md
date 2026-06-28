# Permission Matrix

> All **265 permissions** declared across §11 of the 30 module specs, namespaced `module.entity.action`. Permissions are enforced through the 3-layer model (RBAC role → scope → ownership; architecture D35).

| Mod | Module | # Perms | Permissions |
|---|---|---|---|
| 01 | Authentication Module | 9 | `auth.audit.export`, `auth.policy.manage`, `auth.session.revoke`, `auth.session.view`, `auth.user.deactivate`, `auth.user.invite`, `auth.user.reactivate`, `auth.user.unlock`, `auth.user.view` |
| 02 | Authorization Module | 9 | `authz.change.approve`, `authz.matrix.export`, `authz.membership.grant`, `authz.membership.revoke`, `authz.membership.view`, `authz.registry.manage`, `authz.role.manage`, `authz.role.view`, `authz.sod.manage` |
| 03 | Institute Module | 10 | `institute.activate`, `institute.archive`, `institute.config.export`, `institute.config.manage`, `institute.create`, `institute.reactivate`, `institute.summary.view`, `institute.suspend`, `institute.update`, `institute.view` |
| 04 | Campus Module | 10 | `campus.config.manage`, `campus.create`, `campus.deactivate`, `campus.default.set`, `campus.export`, `campus.import`, `campus.transfer.approve`, `campus.transfer.execute`, `campus.update`, `campus.view` |
| 05 | Academic Session Module | 10 | `session.activate`, `session.change.approve`, `session.close`, `session.create`, `session.export`, `session.reopen`, `session.rollover`, `session.structure.instance`, `session.update`, `session.view` |
| 06 | Admission Module | 8 | `admission.application.evaluate`, `admission.application.view`, `admission.config.manage`, `admission.convert`, `admission.decision.approve`, `admission.export`, `admission.import`, `admission.waitlist.manage` |
| 07 | Student Module | 12 | `student.anonymize`, `student.archive`, `student.create`, `student.document.manage`, `student.dynamic.update`, `student.export`, `student.guardian.manage`, `student.import`, `student.official.update`, `student.status.manage`, `student.subject_access`, `student.view` |
| 08 | Enrollment Module | 10 | `enrollment.create`, `enrollment.export`, `enrollment.import`, `enrollment.promote`, `enrollment.rollnumber.manage`, `enrollment.transfer.approve`, `enrollment.transfer.execute`, `enrollment.update`, `enrollment.view`, `enrollment.withdraw` |
| 09 | Guardian Module | 10 | `guardian.change.approve`, `guardian.consent.manage`, `guardian.create`, `guardian.export`, `guardian.guardianship.update`, `guardian.import`, `guardian.link.manage`, `guardian.restriction.manage`, `guardian.role.manage`, `guardian.view` |
| 10 | Attendance Module | 9 | `attendance.config.manage`, `attendance.correct`, `attendance.correction.approve`, `attendance.export`, `attendance.holiday.manage`, `attendance.mark`, `attendance.override`, `attendance.report.view`, `attendance.view` |
| 11 | Class Module | 10 | `class.capacity.manage`, `class.change.approve`, `class.create`, `class.export`, `class.import`, `class.promotion.manage`, `class.retire`, `class.stream.manage`, `class.update`, `class.view` |
| 12 | Section Module | 8 | `section.classteacher.assign`, `section.create`, `section.export`, `section.import`, `section.restructure`, `section.restructure.approve`, `section.update`, `section.view` |
| 13 | Subject Module | 10 | `subject.component.manage`, `subject.create`, `subject.curriculum.manage`, `subject.elective.manage`, `subject.export`, `subject.import`, `subject.retire`, `subject.teacher.assign`, `subject.update`, `subject.view` |
| 14 | Examination Module | 9 | `exam.config.manage`, `exam.correction.approve`, `exam.eligibility.manage`, `exam.marks.enter`, `exam.marks.submit`, `exam.moderation.manage`, `exam.report.view`, `exam.view`, `report.export` |
| 15 | Result Processing Module | 8 | `report.export`, `result.compute`, `result.config.manage`, `result.publish`, `result.revise`, `result.revision.approve`, `result.view`, `result.withhold.manage` |
| 16 | Grading Module | 6 | `grading.change.approve`, `grading.config.manage`, `grading.export`, `grading.import`, `grading.override`, `grading.view` |
| 17 | Fee Management Module | 9 | `fee.config.manage`, `fee.creditnote.issue`, `fee.export`, `fee.fine.manage`, `fee.import`, `fee.invoice.issue`, `fee.refund.approve`, `fee.refund.process`, `fee.view` |
| 18 | Payment Module | 9 | `payment.config.manage`, `payment.correct`, `payment.export`, `payment.import`, `payment.reconcile`, `payment.record`, `payment.reversal.approve`, `payment.reverse`, `payment.view` |
| 19 | Discount Module | 8 | `discount.apply`, `discount.approve`, `discount.config.manage`, `discount.export`, `discount.import`, `discount.reverse`, `discount.view`, `discount.waiver.apply` |
| 20 | Scholarship Module | 9 | `scholarship.approve`, `scholarship.config.manage`, `scholarship.disburse`, `scholarship.export`, `scholarship.import`, `scholarship.lifecycle.manage`, `scholarship.select`, `scholarship.sponsor.view`, `scholarship.view` |
| 21 | Teacher Module | 7 | `academic.assignment.manage`, `section.classteacher.assign`, `teacher.designate`, `teacher.export`, `teacher.import`, `teacher.qualification.manage`, `teacher.workload.approve` |
| 22 | Staff Module | 10 | `hr.separation.manage`, `staff.account.link`, `staff.change.approve`, `staff.export`, `staff.hierarchy.manage`, `staff.import`, `staff.manage`, `staff.scope.manage`, `staff.status.manage`, `staff.view` |
| 23 | HR Module | 11 | `hr.config.manage`, `hr.export`, `hr.import`, `hr.lifecycle.manage`, `hr.settlement.manage`, `hr.view`, `payroll.approve`, `payroll.compute`, `payroll.config.manage`, `payroll.disburse`, `payroll.salary.approve` |
| 24 | Leave Management Module | 8 | `leave.approve`, `leave.backdate`, `leave.balance.manage`, `leave.config.manage`, `leave.export`, `leave.import`, `leave.view`, `student.leave.approve` |
| 25 | Notification Module | 6 | `notification.config.manage`, `notification.delivery.manage`, `notification.export`, `notification.report.view`, `notification.status.view`, `notification.template.manage` |
| 26 | Reporting Module | 8 | `report.audit.view`, `report.bulk_pii.approve`, `report.cross_institute.view`, `report.definition.manage`, `report.export`, `report.run`, `report.schedule.manage`, `report.subject_access` |
| 27 | Configuration Engine Module | 9 | `config.definition.manage`, `config.export`, `config.import`, `config.rollback`, `config.schema.manage`, `config.sensitive.manage`, `config.value.approve`, `config.value.manage`, `config.view` |
| 28 | Workflow Engine Module | 8 | `workflow.act`, `workflow.bulk.act`, `workflow.definition.manage`, `workflow.delegate`, `workflow.instance.initiate`, `workflow.instance.manage`, `workflow.report.view`, `workflow.view` |
| 29 | Audit Log Module | 7 | `audit.erasure.manage`, `audit.export`, `audit.legal_hold.manage`, `audit.meta.read`, `audit.policy.manage`, `audit.read`, `audit.verify` |
| 30 | File Management Module | 8 | `file.access`, `file.config.manage`, `file.generate`, `file.quarantine.manage`, `file.report.view`, `file.retention.manage`, `file.upload`, `file.version.manage` |

## Permission namespace roots (governance surface)
|Namespace root|# permissions|
|---|---|
| `student.*` | 13 |
| `institute.*` | 10 |
| `campus.*` | 10 |
| `session.*` | 10 |
| `enrollment.*` | 10 |
| `guardian.*` | 10 |
| `class.*` | 10 |
| `subject.*` | 10 |
| `report.*` | 10 |
| `auth.*` | 9 |
| `authz.*` | 9 |
| `attendance.*` | 9 |
| `section.*` | 9 |
| `fee.*` | 9 |
| `payment.*` | 9 |
| `scholarship.*` | 9 |
| `staff.*` | 9 |
| `config.*` | 9 |
| `admission.*` | 8 |
| `exam.*` | 8 |
| `discount.*` | 8 |
| `workflow.*` | 8 |
| `file.*` | 8 |
| `result.*` | 7 |
| `hr.*` | 7 |
| `leave.*` | 7 |
| `audit.*` | 7 |
| `grading.*` | 6 |
| `notification.*` | 6 |
| `teacher.*` | 5 |
| `payroll.*` | 5 |
| `academic.*` | 1 |

## Separation-of-Duties surface — `.approve` permissions (24)

These are the distinct approval authorities that must be held by actors **other** than the requester/initiator (enforced by the Workflow Engine, AUTHZ-009):

`admission.decision.approve`, `attendance.correction.approve`, `authz.change.approve`, `campus.transfer.approve`, `class.change.approve`, `config.value.approve`, `discount.approve`, `enrollment.transfer.approve`, `exam.correction.approve`, `fee.refund.approve`, `grading.change.approve`, `guardian.change.approve`, `leave.approve`, `payment.reversal.approve`, `payroll.approve`, `payroll.salary.approve`, `report.bulk_pii.approve`, `result.revision.approve`, `scholarship.approve`, `section.restructure.approve`, `session.change.approve`, `staff.change.approve`, `student.leave.approve`, `teacher.workload.approve`