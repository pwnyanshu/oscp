class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
        
class LL:
    def __init__(self):
        self.head = None
    
    def insert_at_beg(self, data):
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node
        
    def insert_at_end(self, data):
        new_node = Node(data)
        
        if not self.head:
            self.head = new_node
            new_node.next = None
        
        temp = self.head
        while temp:
            temp1 = temp
            temp = temp.next
        
        temp1.next = new_node
        new_node.next = None
    
        
    def insert_at_pos(self, pos, data):
        temp = self.head
        new_node = Node(data)
        cur = 0
        while cur < pos:
            cur = cur + 1
            temp1 = temp
            temp = temp.next
        temp1.next = new_node
        new_node.next = temp
            

one = LL()

one.insert_at_end(1)
one.insert_at_end(2)
one.insert_at_end(3)
one.insert_at_end(4)
one.insert_at_end(5)
one.insert_at_pos(3, 3.5)