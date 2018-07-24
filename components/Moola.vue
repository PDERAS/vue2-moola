<template>
    <input ref="moola"
           type="text"
           :value="formattedValue"
           @input="formatInput"
           @keydown="checkKeys($event)"
           @blur="blur($event)"
           @mousemove.prevent
           @focusin="highlight"
           @focus="stripFormatters" >
</template>

<script>
    export default {
        name: 'moola',

        props: {
            delimiter: {
                type: String,
                default: ','
            },

            max: {
                type: Number,
                default: Number.MAX_SAFE_INTEGER
            },

            min: {
                type: Number,
                default: Number.MIN_SAFE_INTEGER
            },

            /**
             * Sets if the base value can be null
             * or defaults to 0.
             *
             * @var boolean
             */
            nullable: {
                type: Boolean,
                default: false
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
                    /* Don't format value if it is null and prop is true */
                    if (this.nullable && (newVal === null || newVal === '')) {
                        this.formattedValue = null;
                        return newVal;
                    }
                    var el = this.$refs['moola'];
                    var formatted = this.addDelimiters(newVal);

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
            /* Format value if it is not null and prop is true */
            if (this.nullable && (this.value !== null && this.value !== '')) {
                var formatted = this.addDelimiters(Number(this.value).toFixed(this.precision));
                if (this.prefix) {
                    formatted = this.prefix + formatted;
                }
                if (this.suffix) {
                    formatted = formatted + this.suffix;
                }
                this.formattedValue = formatted;
            }
        },

        methods: {
            addDelimiters(val) {
                val = String(val);

                var split = val.split('.');

                while (/(\d+)(\d{3})/.test(split[0])) {
                    split[0] = split[0].replace(/(\d+)(\d{3})/, '$1' + this.delimiter + '$2');
                }

                return split.join('.');
            },

            blur(event) {
                /* Short circuit function if input is cleared */
                if (this.nullable && event.target.value === '') {
                    this.$emit('input', null);
                    this.formattedValue = null;
                    return;
                }
                var val = this.removeDelimiters(event.target.value);
                var split = val.split('.');
                var el = this.$refs['moola'];

                var emitVal = Number(val).toFixed(this.precision);

                if (isNaN(emitVal) || !isFinite(emitVal)) {
                    emitVal = "0.00";
                }
                el.value = this.addDelimiters(emitVal);
                this.$emit('input', emitVal);

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
                var keyCode = event.which || event.keyCode;
                var otherKey = event.metaKey || event.altKey || event.ctrlKey;

                var ref = this.$refs['moola'];
                var val = String(ref.value);

                var cursorPosition = ref.selectionStart;
                var selectionSize = ref.selectionEnd - ref.selectionStart;

                /* Functional, Navigational and Number Keys */
                if (otherKey
                    || (keyCode < 48 && keyCode != 32 & keyCode != 8)
                    || (keyCode > 48 && keyCode <= 57)
                    || (keyCode >= 91 && keyCode <= 93)
                    || (keyCode > 96 && keyCode <= 105)
                    || (keyCode >= 112 && keyCode <= 145)) {
                    return;
                } else {
                    /* Backspace */
                    if (keyCode == 8) {
                        if (String(val)[cursorPosition - 1] == this.delimiter) {
                            var newVal = val.slice(0, cursorPosition - 2) + val.slice(cursorPosition, val.length);
                            this.$emit('input', newVal);
                            event.preventDefault();
                        }
                    /* Period or Decimal */
                    } else if ((keyCode == 110 || keyCode == 190) && this.precision > 0) {
                        var newVal = String(val) + '.';
                        var split = newVal.split('.');

                        if (split.length > 2) {
                            event.preventDefault();
                        }
                    /* Zero */
                    } else if (keyCode == 48 || keyCode == 96) {
                        if (cursorPosition == 0 && String(val).length > 0 && String(val[0]) != '.' && selectionSize == 0) {
                            event.preventDefault();
                        }
                    /* Minus */
                    } else if (keyCode == 109 || keyCode == 189) {
                        if (cursorPosition != 0) {
                            if (String(val[0]) == '-') {
                                this.$emit('input', String(this.value).substring(1));
                            } else {
                                this.$emit('input', '-' + this.value);
                            }
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
                var val = this.removeDelimiters(event.target.value);
                var split = val.split('.');
                var el = this.$refs['moola'];
                var positionFromEnd = el.value.length - el.selectionEnd;
                var offset = 0;

                if (split[0].length == 2 && split[0][0] == '0') {
                    val = String(val).substr(1, String(val).length - 1);
                }

                if (split.length > 1 && split[1].length > this.precision) {
                    var lastDigit = positionFromEnd < 2 ? split[1][split[1].length - 1 - positionFromEnd] : split[1][split[1].length - 1];
                    val = String(val).substr(0, String(val).length - 2) + lastDigit;
                    offset = 1;
                }

                if (Number(val) > this.max) {
                    val = this.max.toFixed(this.precision);
                }

                if (Number(val) < this.min) {
                    val = this.min.toFixed(this.precision);
                }
                el.value = this.addDelimiters(val);

                positionFromEnd = el.value.length - positionFromEnd + offset;

                this.$emit('input', val);
                if (positionFromEnd < 0) {
                    positionFromEnd = 0;
                }
                // Nothing has been highlighted...
                if (el.selectionStart == el.selectionEnd) {
                    this.setCursor(el, positionFromEnd);
                }
            },

            removeDelimiters(val) {
                return (String(val).split(this.delimiter)).join('');
            },

            setCursor(el, position) {
                var setSelectionRange = function () { el.setSelectionRange(position, position) }
                if (el === document.activeElement) {
                    setSelectionRange()
                }
            },

            stripFormatters(event) {
                /* Only strip formatting if a value is set */
                if (event.target.value !== null && event.target.value !== '') {
                    this.stripPrefix(event);
                    this.stripSuffix(event);
                }
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


