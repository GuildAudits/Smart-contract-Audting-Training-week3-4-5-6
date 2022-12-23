# Hash Collisions With Multiple Variable Length Arguments



Using `abi.encodePacked()` with multiple variable length arguments can, in certain situations, lead to a `hash collision`. Since `abi.encodePacked()` packs all elements in order regardless of whether they're part of an array, you can move elements between arrays and, so long as all elements are in the same order, it will return the `same encoding`. In a signature verification situation, an attacker could exploit this by modifying the position of elements in a previous function call to effectively bypass authorization.
