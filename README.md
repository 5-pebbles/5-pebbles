### I am starting to see why these codebases (glibc, musl, etc...) are so hard to read as a beginner:

In programming, we usually rely on abstraction to simplify our jobs. We make structures to contain and compartmentalize complex or dangerous code. However, at this level, those just obfuscate an already confusing system.

You start to find looking up definitions annoying; it's a null terminated list of elements; just treat it that way. On top of that, laziness works in that ideas favor:

```rs
pub(crate) const AT_NULL: usize = 0;
pub(crate) const AT_PAGE_SIZE: usize = 6;
pub(crate) const AT_BASE: usize = 7;
pub(crate) const AT_ENTRY: usize = 9;

#[repr(C)]
#[derive(Clone, Copy)]
pub(crate) struct AuxiliaryVectorItem {
    pub a_type: usize,
    pub a_val: usize,
}

#[derive(Clone, Copy)]
pub(crate) struct AuxiliaryVectorIter(*const AuxiliaryVectorItem);

impl AuxiliaryVectorIter {
    pub(crate) fn new(auxiliary_vector_pointer: *const AuxiliaryVectorItem) -> Self {
        Self(auxiliary_vector_pointer)
    }

    pub(crate) fn into_inner(self) -> *const AuxiliaryVectorItem {
        self.0
    }
}

impl Iterator for AuxiliaryVectorIter {
    type Item = AuxiliaryVectorItem;

    fn next(&mut self) -> Option<Self::Item> {
        let this = unsafe { *self.0 };
        if this.a_type == AT_NULL {
            return None;
        }
        self.0 = unsafe { self.0.add(1) };
        Some(this)
    }
}
```

The truth is you are only going to use that struct 3 or so times; why not just write:

```rs
(0..).map(|i| unsafe { *auxiliary_vector_pointer.add(i) }).take_while(|t| t.a_type != AT_NULL)
```

The same goes for naming; is `AuxiliaryVector` really any more helpful than `auxv`? Many of these things you can only find in pdfs from the 90s; if you change the name to something more descriptive, you run the risk of no one being able to understand you.

Either way, what is a more descriptive name? It's just an assortment of possibly useful stuff passed to the linker by the Linux kernel... You are going to have to look it all up anyway.
