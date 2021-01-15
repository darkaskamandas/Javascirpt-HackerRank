var BigNumberConstructor = require("bignumber.js");
const cache = {};
const createGraphRepresentation = (src) => {
    var id = src;
    if (!cache[id]) {
        var num = new BigNumberConstructor(src);
        cache[id] = new GraphRepresentation(num);
    }
    return cache[id];
};
class GraphRepresentation {
    constructor(number) {
        this.bignumber = number;
        this.id = number.toString();
    }
    get binary() {
        if (!this.binaryData) {
            this.binaryData = new Uint8Array(this.bignumber.toString(2).split("").map(Number));
        }
        return this.binaryData;
    }
    get verticesNum() {
        if (!this.verticesData) {
            var counter = 0;
            for (let c of this.binary) {
                if (c === 1) {
                    counter++;
                }
            }
            this.verticesData = counter;
        }
        return this.verticesData;
    }
    intersects(other) {
        if (!this.intersectsCache) {
            this.intersectsCache = new Map();
        }
        var cached = this.intersectsCache.get(other);
        if (cached) {
            return cached;
        }
        var thisBinary = this.binary;
        var otherBinary = other.binary;
        var l = Math.min(thisBinary.length, otherBinary.length);
        var retval = false;
        for (var i = 1; i <= l; i++) {
            if (thisBinary[thisBinary.length - i] === 1 && otherBinary[otherBinary.length - i] === 1) {
                retval = true;
                break;
            }
        }
        this.intersectsCache.set(other, retval);
        if (!other.intersectsCache) {
            other.intersectsCache = new Map();
        }
        other.intersectsCache.set(this, retval);
        return retval;
    }
    union(other) {
        var thisBinary = this.binary;
        var otherBinary = other.binary;
        if (!this.unionCache) {
            this.unionCache = new Map();
        }
        const cached = this.unionCache.get(other);
        if (!cached) {
            var l = Math.max(thisBinary.length, otherBinary.length);
            var res = [];
            for (var i = 1; i <= l; i++) {
                var a = i > thisBinary.length ? 0 : thisBinary[thisBinary.length - i];
                var b = i > otherBinary.length ? 0 : otherBinary[otherBinary.length - i];
                if (a === 1 || b === 1) {
                    res[l - i] = 1;
                }
                else {
                    res[l - i] = 0;
                }
            }
            var retval = createGraphRepresentation(new BigNumberConstructor(res.join(""), 2).toString());
            this.unionCache.set(other, retval);
            if (!other.unionCache) {
                other.unionCache = new Map();
            }
            other.unionCache.set(this, retval);
            return retval;
        }
        return cached;
    }
    equal(other) {
        return this.bignumber === other.bignumber;
    }
}
const getVerticesNum = (d) => {
    return d.toString(2).split("1").length - 1;
};
const intersectsWithNone = new Map();
const processData = (input) => {
    var reps = input.split(" ").map((a) => {
        return createGraphRepresentation(a);
    });
    loop: for (let r1 of reps) {
        let intersectsCount = 0;
        for (let r2 of reps) {
            if (r1.intersects(r2)) {
                intersectsCount++;
                if (intersectsCount > 1) {
                    continue loop;
                }
            }
        }
        intersectsWithNone.set(r1, true);
    }
    const subsets = getSubstets(reps);
    var res = 0;
    for (let subset of subsets) {
        //console.log(subset[0]);
        var subsetRes = 64;
        for (let d of subset) {
            //console.log("look at ", d);
            subsetRes -= Math.max(0, d.verticesNum - 1);
        }
        //console.log(subsetRes);
        res += subsetRes;
    }
    return res;
};
const getCombinations = (arr, value) => {
    var a = [];
    loop: for (var k = 0; k < arr.length; k++) {
        if (((value >> k) & 1) > 0) {
            var idx = k;
            if (!intersectsWithNone.has(arr[idx])) {
                for (var i = 0; i < a.length; i++) {
                    if (a[i].intersects(arr[idx])) {
                        a[i] = a[i].union(arr[idx]);
                        continue loop;
                    }
                }
            }
            a.push(arr[idx]);
        }
    }
    return a;
};
const getSubstets = (arr) => {
    var l = Math.pow(2, arr.length);
    var res = [];
    for (var i = 0; i < l; i++) {
        res.push(getCombinations(arr, i));
    }
    return res;
};

process.stdin.resume();
process.stdin.setEncoding("ascii");
_input = "";
process.stdin.on("data", function (input) {
    _input += input;
});

process.stdin.on("end", function () {
   console.log(processData(_input.split("\n")[1]));
});