# Historical Review Lessons

Store concise, generalized lessons from past human reviews here. Keep only guidance
that can prevent a recurring issue; do not archive raw review threads.

## Architecture

No repository-specific lessons recorded yet.

## Correctness and Delivery

No repository-specific lessons recorded yet.


# Historical Human Review Findings

## Finding 001

**[P1] 非成功 Outcome 仍能携带 Withdrawal**

`validation.py` L286-L321 只校验 Withdrawal 的排序和 Reason Code，没有限制 Outcome 状态。

我实际构造以下组合，Validator 全部返回 accepted：

```text
rejected    + proactive.contract_invalid     + withdrawal
quarantined + proactive.policy_unsupported   + withdrawal
duplicate   + proactive.evaluation_duplicate + withdrawal
