Solution1:  
A totally bad solution!!
```python
def part_of_float(character):
    return character.isdigit() or character == "."

def flip_string_number(number):
    if number[0] == "-":
        number = number[1 : : ]
    else:
        number = "-" + number
    return number


def parse_expstring_to_items(expstring):
    expstring = expstring.replace(" ", "")
    res = []
    is_negative = False
    tmp_str = ""
    for ele in expstring:
        if ele == "-":
            # negative: len(tmp_str)==0 and (len(res) == 0 or res[-1] in +-*/(-( action: set negative flag
            # minus: len(tmp_str)>0 or res[-1]==),  action: flush tmp_str with negative flag, res.append("-")
            if len(tmp_str) == 0 and (len(res) == 0 or res[-1] in {"+", "-", "*", "/", "(", "-("}):
                is_negative = True
            elif len(tmp_str) > 0 or res[-1] == ")":
                if len(tmp_str) > 0:
                    if is_negative == True:
                        tmp_str = flip_string_number(tmp_str)
                        is_negative = False
                    res.append(tmp_str)      
                    tmp_str = ""
                res.append("-")
        elif ele == "(":
            if is_negative == True:
                ele = "-" + ele
                is_negative = False
            res.append(ele)           
        elif not part_of_float(ele):
            if len(tmp_str) != 0:
                if is_negative == True:
                    tmp_str = flip_string_number(tmp_str)
                    is_negative = False
                res.append(tmp_str)
                tmp_str = ""
            res.append(ele)
        else:
            tmp_str += ele
    if len(tmp_str) != 0:
        if is_negative == True:
            tmp_str = flip_string_number(tmp_str)
        res.append(tmp_str)
        tmp_str == ""
    return res


def number_map(eles):
    number_map = []
    for ele in eles:
        try:
            float(ele)
            number_map.append(True)
        except:
            number_map.append(False)
    return number_map


def grade(operator):
    if operator in {"(", "-("}:
        return 0
    elif operator in {"+", "-"}:
        return 1
    elif operator in {"*", "/"}:
        return 2
    elif operator in {")"}:
        return 3


def grade_compare(ope1, ope2):
    return grade(ope1) - grade(ope2)


def do_arithmetic(right, ope, left):
    left = float(left)
    right = float(right)
    res = None
    if ope == "+":
        res = left + right
    elif ope == "-":
        res = left - right
    elif ope == "*":
        res = left * right
    elif ope == "/":
        res = left / right
    return str(res)


def popback_tmpstack(tmpstack, operands, operators):
    while tmpstack and tmpstack[-1] != ")":
        data = tmpstack.pop()
        if data in {"+", "-", "*", "/"}:
            operators.append(data)
        else:
            operands.append(data)


def opposite_ope(ope):
    if ope == "+":
        return "-"
    elif ope == "-":
        return "+"
    elif ope == "*":
        return "/"
    elif ope == "/":
        return "*"


# the key problem here is:
# 1. right to left calculation
# 2. when can i go on with one calculation and when should i just temp store them for higher grade actions


def eval_with_stack(operands, operators):
    helper = []
    while operators:
        ope_fir = operators.pop()
        if ope_fir in {"+", "-"}:
            if operators and grade_compare(ope_fir, operators[-1]) < 0:
                helper.append(operands.pop())
                helper.append(ope_fir)
            else:
                if operators and operators[-1] == "-":
                    ope_fir = opposite_ope(ope_fir)
                res = do_arithmetic(operands.pop(), ope_fir, operands.pop())
                operands.append(res)
                popback_tmpstack(helper, operands, operators)
        elif ope_fir in {"*", "/"}:
            if operators and grade_compare(ope_fir, operators[-1]) < 0:
                helper.append(operands.pop())
                helper.append(ope_fir)
            else:
                if operators and operators[-1] == "/":
                    ope_fir = opposite_ope(ope_fir)
                res = do_arithmetic(operands.pop(), ope_fir, operands.pop())
                operands.append(res)
                popback_tmpstack(helper, operands, operators)
        elif ope_fir in {")"}:
            helper.append(ope_fir)
        elif ope_fir in {"(", "-("}:
            if helper[-1] == ")":
                helper.pop()
                if ope_fir == "-(":
                    operands.append(flip_string_number(operands.pop()))
                popback_tmpstack(helper, operands, operators)
            # else:
            #     popback_tmpstack(helper, operands, operators)

    return float(operands.pop())


def my_eval(string):
    eles = parse_expstring_to_items(string)
    number_map_var = number_map(eles)
    print(eles)
    print(number_map_var)
    operands = []
    operators = []
    for ele, is_number in zip(eles, number_map_var):
        if is_number:
            operands.append(ele)
        else:
            operators.append(ele)
    return eval_with_stack(operands, operators)


def calc(expression):
    print("=========="+expression)
    return my_eval(expression)


if __name__ == "__main__":
    s1 = "-(3 + -(6+3)*4 - -(-3/3 + 2*7))"
    r1 = 20
    # s1 = "-(-6/3 + -18/(2+7)*4.0)"
    # r1 = 2.5
    # s1 = "-(3 + -(6+3.9)*90.4 - -(-3/6.8 + 2.893*7.235))"
    # r1 = -1208.76
    s1 = "(3)--3"
    r1 = 12
    res = calc(s1)
    print(res)
    # assert res == r1


    # ["2 + -2", 0],
    # ["10- 2- -5", 13],
```

Solution2:
```python
class MathExpressionEval:
    def __init__(self, expression):
        self.expression = expression
        self.token_list = None
        self.token_tree = None

    def part_of_float(self, character):
        return character.isdigit() or character == "."


    def parse_tokens(self):
        self.expression = self.expression.replace(" ", "")
        res = []
        tmp_str = ""
        for ele in self.expression:
            if self.part_of_float(ele):
                tmp_str = tmp_str + ele
            else:
                if tmp_str != "":
                    res.append(tmp_str)
                    tmp_str = ""
                # should be (+_*/ here, need to pick up negative
                if ele == "-" and (not res or not self.part_of_float(res[-1]) and res[-1] != ")"):
                    res.append("n")
                else:
                    res.append(ele)
        if tmp_str != "":
            res.append(tmp_str)
        print(res)
        return res


    def number_map(self, eles):
        number_map = []
        for ele in eles:
            try:
                float(ele)
                number_map.append(True)
            except:
                number_map.append(False)
        return number_map


    def grade(self, operator):
        if operator in {"(", ")"}:
            return 1
        elif operator in {"+", "-"}:
            return 2
        elif operator in {"n"}:
            return 3
        elif operator in {"*", "/", "%"}:
            return 4


    def do_arithmetic(self, ope, right, left=None):
        res = None
        right = float(right)
        if left == None and ope == "n":
            res = -1 * right
        else:
            left = float(left)
            if ope == "+":
                res = left + right
            elif ope == "-":
                res = left - right
            elif ope == "*":
                res = left * right
            elif ope == "/":
                res = left / right
            elif ope == "%":
                res = left % right
        return str(res)


    def my_eval(self):
        self.token_list = self.parse_tokens()


def calc(expression):
    print(expression)
    e = MathExpressionEval(expression)
    return e.my_eval()


if __name__ == "__main__":
    tests = [
    # ["1 + 1", 2],
    # ["8/16", 0.5],
    # ["3 -(-1)", 4],
    # ["2 + -2", 0],
    ["10- 2- -5", 13],
    ["(((10)))", 10],
    ["3 * 5", 15],
    ["-7 * -(6 / 3)", 14], 
    ["-(1 + -(3+2)*4 - -(-5/10 + 9/6))",18], 
]

for test in tests: 
    res = calc(test[0])
    print(f"result is : {res}")
    # assert  res == test[1]
```