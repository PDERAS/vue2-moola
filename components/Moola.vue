<template>
    <input ref="moola"
           type="text"
           :value="formattedValue"
           @input="formatInput"
           @keydown="checkKeys($event)"
           @blur="checkDecimals($event)"
           @mousemove.prevent
           @focusin="highlight"
           @focus="stripFormatters" >
</template>

<script>
    export default {
        name: 'moola',

        props: {
            max: {
                type: Number,
                default: Number.MAX_SAFE_INTEGER
            },

            min: {
                type: Number,
                default: Number.MIN_SAFE_INTEGER
            },

            precision: {
                type: Number,
                default: 2
            },

            prefix: {
                type: String,
                default: '$'
            },

            suffix: {
                type: String,
                default: null
            },

            value: {
                type: [ Number, String ],
                default: 0
            }
        },

        data: () => ({
            formattedValue: ''
        }),

        watch: {
            value: {
                handler(newVal) {
                    var el = this.$refs['moola'];
                    var formatted = this.addCommas(newVal);

                    if (el && el !== document.activeElement) {
                        if (this.prefix) {
                            formatted = this.prefix + formatted;
                        }
                        if (this.suffix) {
                            formatted = formatted + this.suffix;
                        }
                    }
                    if (formatted !== this.formattedValue) {
                        this.formattedValue = formatted;
                    }
                    return Number(newVal);
                },
                immediate: true
            }
        },

        mounted() {
            var formatted = this.addCommas(Number(this.value).toFixed(this.precision));
            if (this.prefix) {
                formatted = this.prefix + formatted;
            }
            if (this.suffix) {
                formatted = formatted + this.suffix;
            }
            this.formattedValue = formatted
        },

        methods: {
            addCommas(val) {
                val = String(val);

                var split = val.split('.');

                while (/(\d+)(\d{3})/.test(split[0])) {
                    split[0] = split[0].replace(/(\d+)(\d{3})/, '$1' + ',' + '$2');
                }

                return split.join('.');
            },

            checkDecimals(event) {
                var val = this.removeCommas(event.target.value);
                var split = val.split('.');
                var el = this.$refs['moola'];

                el.value = this.addCommas(Number(val).toFixed(this.precision));
                this.$emit('input', Number(val).toFixed(this.precision));

                if (el && el !== document.activeElement) {
                    if (this.prefix) {
                        this.formattedValue = this.prefix + this.formattedValue;
                    }
                    if (this.suffix) {
                        this.formattedValue = this.formattedValue + this.suffix;
                    }
                }
            },

            checkKeys(event) {
                this.stripFormatters(event);
                var keyCode = event.keyCode;
                var otherKey = event.metaKey || event.altKey || event.ctrlKey;

                var ref = this.$refs['moola'];
                var val = String(ref.value);

                var cursorPosition = ref.selectionStart;
                var selectionSize = ref.selectionEnd - ref.selectionStart;

                /* Functional, Navigational and Number Keys */
                if (otherKey
                    || (keyCode < 48 && keyCode != 32)
                    || (keyCode > 48 && keyCode <= 57)
                    || (keyCode >= 91 &&  keyCode <= 93)
                    || (keyCode > 96 &&  keyCode <= 105)
                    || (keyCode >= 112 && keyCode <= 145)) {
                    return;
                } else {
                    /* Period or Decimal */
                    if ((keyCode == 110 || keyCode == 190) && this.precision > 0) {
                        var newVal = String(val) + '.';
                        var split = newVal.split('.');

                        if (split.length > 2) {
                            event.preventDefault();
                        }
                    /* Zero */
                    } else if (keyCode == 48 || keyCode == 96) {
                        if (cursorPosition == 0 && String(val).length > 0 && String(val[0]) != '.' && selectionSize == 1) {
                            event.preventDefault();
                        }
                    } else {
                        event.preventDefault();
                    }
                }
            },

            highlight(event) {
                var el = this.$refs['moola'];
                el.select();
            },

            formatInput(event) {
                var val = this.removeCommas(event.target.value);
                var split = val.split('.');
                var el = this.$refs['moola'];
                var positionFromEnd = el.value.length - el.selectionEnd;
                var decimalOffset = 0;

                if (split[0].length == 2 && split[0][0] == '0') {
                    val = String(val).substr(1, String(val).length - 1);
                }

                if (split.length > 1 && split[1].length > this.precision) {
                    var lastDigit = positionFromEnd < 2 ? split[1][split[1].length - 1 - positionFromEnd] : split[1][split[1].length - 1];
                    val = String(val).substr(0, String(val).length - 2) + lastDigit;
                    decimalOffset = 1;
                }


                if (Number(val) > this.max) {
                    val = this.max.toFixed(this.precision);
                }

                if (Number(val) < this.min) {
                    val = this.min.toFixed(this.precision);
                }
                el.value = this.addCommas(val);
                positionFromEnd = el.value.length - positionFromEnd + decimalOffset;

                this.$emit('input', val);

                // Nothing has been highlighted...
                if (el.selectionStart == el.selectionEnd) {
                    this.setCursor(el, positionFromEnd);
                }
            },

            removeCommas(val) {
                return (String(val).split(',')).join('');
            },

            setCursor(el, position) {
                var setSelectionRange = function () { el.setSelectionRange(position, position) }
                if (el === document.activeElement) {
                    setSelectionRange()
                }
            },

            stripFormatters(event) {
                this.stripPrefix(event);
                this.stripSuffix(event);
            },

            stripPrefix(event) {
                event.target.value = event.target.value.replace(this.prefix, '');
                this.formattedValue = this.formattedValue.replace(this.prefix, '');
            },

            stripSuffix(event) {
                event.target.value = event.target.value.replace(this.suffix, '');
                this.formattedValue = this.formattedValue.replace(this.suffix, '');
            },
        }
    }
</script>


