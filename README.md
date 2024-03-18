# Javascript Struct Factory

### An interface for managing raw bytes containing C structs

This is not a formal library, it's more of a snippet.

```js

const StructFactory = function(fieldsToRegister) {
    /* fieldsToRegister=[ { name:..., type:..., }, ... ] */

    // Internal implementation.
    const availableFields = [
        {type: 'char', size: 1, viewtype: Uint8Array},
        {type: 'int32_t', size: 4, viewtype: Int32Array},
        {type: 'uint32_t', size: 4, viewtype: Uint32Array},
        {type: 'int64_t', size: 8, viewtype: BigInt64Array},
        {type: 'uint64_t', size: 8, viewtype: BigUint64Array},
        {type: 'float', size: 4, viewtype: Float32Array},
        {type: 'double', size: 8, viewtype: Float64Array},
    ]
    const accessPoints = new Map();
    let byteOffset = 0, largestElementSize = 0;
    for(let i=0; i<fieldsToRegister.length; i++) {
        const field = fieldsToRegister[i];
        const fieldType = availableFields.find(af => af.type === field.type);
        if(!fieldType) throw new Error(`Unknown field type: ${field.type}`);
        accessPoints.set(field.name, {byteOffset, ...fieldType});
        byteOffset += fieldType.size;
        largestElementSize = Math.max(largestElementSize, fieldType.size);
    }
    let structSize = byteOffset;
    if(structSize % largestElementSize !== 0)
        structSize += (largestElementSize - (structSize % largestElementSize));

    // Public interface.
    function getFieldValue(buffer, ptr, fiendName, elementCt=1){
        /* Be careful when calling this function,
            particularly when elementCt > 1.
        */
        if(!accessPoints.has(fiendName)) throw new Error(`Unknown field name: ${fiendName}`);
        const accessPoint = accessPoints.get(fiendName);
        // console.log('accessPoint', accessPoint);
        const v = new accessPoint.viewtype(buffer, ptr + accessPoint.byteOffset, elementCt);
        return elementCt == 1 ? v[0]: v;
    }
    return {
        getFieldValue,
        structSize,
    };
}


//  UNIT TESTS ///
//
//
//
//
const assertTrue = (condition, message) => {
    if(!condition) throw new Error("AssertionError " + message);
}
const _StructFactoryTests = [
    // Struct Size tests.
    function testSizeIsCalculatedProperlyAllItemsSameSize() {
        const sf = new StructFactory([
            {name: 'a', type: 'int32_t'},
            {name: 'b', type: 'int32_t'},
            {name: 'c', type: 'uint32_t'},
            {name: 'd', type: 'uint32_t'},
            {name: 'e', type: 'uint32_t'},
        ]);
        assertTrue(sf.structSize === 20, `sf.structSize=${sf.structSize}`);
    },
    function testSizeIsCalculatedPropertyDifferentSizedElementsRoundUpRequired() {
        const sf = new StructFactory([
            {name: 'a', type: 'int32_t'},  // 4
            {name: 'b', type: 'int32_t'},  // 8
            {name: 'e', type: 'uint64_t'}, // 16
            {name: 'c', type: 'uint32_t'}, // 20
                                           // +4 padding = 24
        ]);
        assertTrue(sf.structSize === 24, `sf.structSize=${sf.structSize}`);
    },
    function testSizeIsCalculatedPropertyDifferentSizedElementsNoRoundUpRequired() {
        const sf = new StructFactory([
            {name: 'a', type: 'int32_t'},  // 4
            {name: 'b', type: 'int32_t'},  // 8
            {name: 'e', type: 'uint64_t'}, // 16
                                           // +0 padding = 16
        ]);
        assertTrue(sf.structSize === 16, `sf.structSize=${sf.structSize}`);
    },

    // Data access tests.
    function testGetFirstFewUnsignedInt32FieldsUsingTowCalls() {
        const sf = new StructFactory([
            {name: 'a', type: 'uint32_t'},
            {name: 'b', type: 'uint32_t'},
        ]);
        const buffer = new ArrayBuffer(Uint32Array.BYTES_PER_ELEMENT * 10);
        const view = new Uint32Array(buffer, 0, 10);
        view[0] = 1337;
        view[1] = 42069;
        const testA = sf.getFieldValue(buffer, 0, 'a');
        const testB = sf.getFieldValue(buffer, 0, 'b');
        assertTrue(testA === 1337, `testA=${testA}`);
        assertTrue(testB === 42069, `testB=${testA}`);
    },

    function testGetFirstFewUnsignedInt32FieldsUsingOneCall() {
        /* Basic pointer arithmetic supported
        */
        const sf = new StructFactory([
            {name: 'a', type: 'uint32_t'},
            {name: 'b', type: 'uint32_t'},
        ]);
        const buffer = new ArrayBuffer(Uint32Array.BYTES_PER_ELEMENT * 10);
        const view = new Uint32Array(buffer, 0, 10);
        view[0] = 1337;
        view[1] = 42069;
        const test = sf.getFieldValue(buffer, 0, 'a', 2);
        assertTrue(test[0] === 1337, `test[0]=${test[0]}`);
        assertTrue(test[1] === 42069, `test[1]=${test[1]}`);
    },
];
(() => {
    _StructFactoryTests.forEach((test, i) => {
        console.log(`Running struct factory tests ${test.name}...`);
        test();
        console.log("test passed!")
    });
})();


```


![](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExcnRlZndtajIxZjNuZWJvazFuNGR4aDU2cGxzd2Rkd2h4MTc0NXhmbSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/wwg1suUiTbCY8H8vIA/giphy-downsized-large.gif)

