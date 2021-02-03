# Results Summary

## Defintions

1. **Access Policy Change** An access policy change is an event that modifies the access policy on a file for a user
2. **Confidentiality** The property regarding reading files that is retained if all most recent access policy changes are respected
3. **Integrity** The property regarding modifying files that is retained if all most recent access policy changes are respected

## Description of the counter examples

1. [Implementation Bug](./implementation_bug_check_not_owner_for_group.jpeg)
    When checking access rights for reading a file, If a user is both the owner of the file and a member of the file's group,
    Only the file's owner permissions are checked. For example if the owner permissions do not allow reads, but the group permissions do, The owner is not allowed to read the file.

2. [Non atomic policy changes](./new_owner_concurrent_to_new_owner_perms.jpeg)
    If the permission changes are non-atomic(given that the access policy is defined by multiple CRDTS), You could get a mixture of behavior. In this example, The owner was changed concurrently to changing the owner's permissions. The outcome is that the new owner gets the new permissions even though the rational outcome would be to get either the old owner with
    the new permissions or the new owner with the old permissions.

3. [Unrelated crdt semantics](./conflicting_semantics_between_unrelated_crdts.jpeg)
    Defining the access policy using multiple unrelated crdts could lead to unexpected behavior even if the access policy changes are atomic. In this example each access policy re-applies any undefined portion of the post-invocation access policy, for example, a change owner changes the owner as well as the owner permissions to already existing permissions. In this example,
    The 2 concurrent access policy changes impose the following.
    1. User10 as owner with no read or write access.
    2. User8 as the owner with read and write access.

    The outcome ends up as User8 as the owner with no permissions, but since User10 is considered a part of `others`, 
    User10 can still read the file, thus violating the first access policy change

4. Accessibility vs Confidentiality
    Assuming that conflicting behavior between access policy CRDTS doesn't occur. There are still some implementation trade-offs to be made regarding how conflicting concurrent access policy changes are resolved. If 2 concurrent access policy changes
    occur where one takes away write permissions from a user but grants read permissions, and the other one takes away read permissions but grants write permissions, there would be 2 ways to resolve this.
    In essence, since both the Owner CRDT and Owner Permissions CRDT are semantically unrelated, One can't define properties that
    address both an owner of file and the owner permissions of the file at the same time. The owner can only be referred to as any owner at the time of checking the property.
    1. [Taking the lower bound](./accessibility_violation.jpeg)
        If only the intersection of the permissions is taken, It's quite possible that one could end up in a situation
        where the entire system is rendered non-accessible even though this was not the intention of any of the concurrent policy change actions.
    2. [Taking the upper bound](./confidentiality_violation)
        If the unions of permissions is taken, One could end up violating confidentiality of one or both of the concurrent policy change.

## Next Steps

- The posix semantics do not fit well in a distributed setting, concurrent changes to access policy might contend on which user is the owner of a file, but the posix semantics do not allow multiple owners. One possible solution is to use Access Control Lists which allows more flexibility and consequently provide a highly tailored setting between consistency and availability.

- Composing access control CRDTs to define a single CRDT that could facilitate handling of non-trivial conflicts.
