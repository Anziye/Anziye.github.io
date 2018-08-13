##代码如下

import sys
import re

def isdigit(com):
    print(com)
    matchObj = re.match(r'([a-zA-Z0-9])-', com)
    # print(matchObj.group(2))
    print(matchObj)
    if matchObj:
        return False
    com = com.replace('-', '')
    if com.isdigit() == True:
        return True
    else:
        return False

def check(road_grid, link):

    if (isdigit(road_grid[0]) == False) or (isdigit(road_grid[1]) == False):
        print("Invalid number format.")
        exit(0)
    if (int(road_grid[0]) <= 0) or (int(road_grid[1]) <= 0):
        print("Number out of range.")
        exit(0)


    for s in link:
        detail = s.split(' ')
        if len(detail) <= 1:
            print("Incorrect command format.")
            exit(0)
        node1 = detail[0]
        node2 = detail[1]
        # print(node1)
        # print(node2)
        node1_detail = node1.split(',')
        node2_detail = node2.split(',')
        print(node1_detail)
        print(node2_detail)
        if (node1_detail[0] != node2_detail[0]) and (node1_detail[1] != node2_detail[1]):
            print("Maze format error.")
            exit(0)





if __name__ == '__main__':
    command1 = input()
    command2 = input()

    road_grid = []
    road_grid = command1.split(' ')
    link = command2.split(';')
    # print(road_grid[0])
    print(link)
    check(road_grid, link)
    h = int(road_grid[0])*2+1
    w = int(road_grid[1])*2+1
    render_grid = [['[w]' for i in range(h)] for i in range(w)]

    for j in range(int(road_grid[0])):
        for k in range(int(road_grid[1])):
            render_grid[2*j+1][2*k+1] = '[R]'

    # for i in render_grid:
    #     print(str(i))
    #
    for l in link:
        detail = l.split(' ')
        node1 = detail[0]
        node2 = detail[1]
        # print(node1)
        # print(node2)
        node1_detail = node1.split(',')
        node2_detail = node2.split(',')
        #print(node1_detail)
        #print(node2_detail)
        h1 = int(node1_detail[0]) * 2 + 1

        w1 = int(node1_detail[1]) * 2 + 1

        h2 = int(node2_detail[0]) * 2 + 1

        w2 = int(node2_detail[1]) * 2 + 1
        # print(h1)
        # print(w1)
        # print(h2)
        # print(w2)

        node3_h = (h1 + h2)/2
        node3_w = (w1 + w2)/2
        # print(node3_w)
        # print(node3_h)
        # print('\n')
        render_grid[int(node3_h)][int(node3_w)] = '[R]'
    for i in render_grid:
        print(str(i))



