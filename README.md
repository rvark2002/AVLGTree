# AVLGTree

An AVLG Tree is an <a href="https://en.wikipedia.org/wiki/AVL_tree">AVL Tree</a> with a relaxed
balance condition. 

The G represents the maximum imbalance of the tree


An AVL-1 tree is a classic AVL tree, which only allows for perfectly balanced binary subtrees (imbalance of 0 everywhere), or subtrees with a
maximum imbalance of 1 (somewhere).

An AVL-2 tree relaxes the criteria of AVL-1 trees, by also allowing for subtrees that have an imbalance of 2

AVL-3 trees allow an imbalance of 3

And so forth...

The idea behind AVL-G trees is that rotations cost time, 
so maybe we would be willing to accept bad search performance now and then if it would mean less rotations. 
On the other hand, increasing the balance parameter also means that we will be making insertions faster.



    public class AVLGTree<T extends Comparable<T>> {


	private int maxImbalance;
	private Node root;

	private class Node {
		private Node lnode;
		private Node rnode;
		private T value;
		private int height;

		public Node() {
			height = 0;
		}
	}

### Constructor
	public AVLGTree(int maxImbalance) throws InvalidBalanceException {
		
		if(maxImbalance < 1)
			throw new InvalidBalanceException("Invalid Balance");
		this.maxImbalance = maxImbalance;

	}

### Insert Key in Tree

	public void insert(T key) {
		
		//Avoid duplicates
		try {
			if(search(key) != null)
				return;
		} catch (EmptyTreeException e) {
			// TODO Auto-generated catch block
		}
		
		if (root == null) {
			// Root is empty
			root = new Node();
			root.value = key;
		} else {
			// Root isnt empty

			root = insertHelper(key, root);

		}

	}

	public Node insertHelper(T key, Node curr) {
		if (curr == null) {
			Node a = new Node();
			a.value = key;
			return a;
		} else {

			if (key.compareTo(curr.value) < 0) {
				// Insert left
				curr.lnode = insertHelper(key, curr.lnode);

			} else {
				// Insert right
				curr.rnode = insertHelper(key, curr.rnode);

			}

			// Rotations

			curr.height = Math.max((heightHelper(curr.lnode)), (heightHelper(curr.rnode)));

			int bal = getBalance(curr);

			// Right Rotation

			if (curr.lnode != null) {
				if (bal > maxImbalance && (key.compareTo(curr.lnode.value) < 0)) {
					return rotateRight(curr);
				}
			}
			// Left Rotation
			if (curr.rnode != null) {
				if (bal < -1*maxImbalance && (key.compareTo(curr.rnode.value) > 0)) {
					return rotateLeft(curr);
				}
			}
			// Left Right Rotation
			if (curr.lnode != null) {
				if (bal > maxImbalance && (key.compareTo(curr.lnode.value) > 0)) {
					curr.lnode = rotateLeft(curr.lnode);
					return rotateRight(curr);
				}
			}

			// Right Left Rotation
			if (curr.rnode != null) {
				if (bal <  -1*maxImbalance && (key.compareTo(curr.rnode.value) < 0)) {
					curr.rnode = rotateRight(curr.rnode);
					return rotateLeft(curr);
				}
			}

			return curr;

		}
	}

	public Node rotateRight(Node target) {
		Node temp = target.lnode;
		target.lnode = temp.rnode;
		temp.rnode = target;

		// Update heights

		temp.rnode.height = Math.max((heightHelper(temp.rnode.lnode)), heightHelper(temp.rnode.rnode));
		temp.height = Math.max(heightHelper(temp.lnode), heightHelper(temp.rnode));
		return temp;
	}

	public Node rotateLeft(Node target) {
		Node temp = target.rnode;
		target.rnode = temp.lnode;
		temp.lnode = target;
		
		temp.lnode.height = Math.max((heightHelper(temp.lnode.lnode)), heightHelper(temp.lnode.rnode));
		temp.height = Math.max(heightHelper(temp.lnode), heightHelper(temp.rnode));
		
		return temp;

	}

	public Node rotateLeftRight(Node target) {
		target.lnode = rotateLeft(target.lnode);
		target = rotateRight(target);
		return target;
	}

	public Node rotateRightLeft(Node target) {
		target.rnode = rotateRight(target.rnode);
		target = rotateLeft(target);
		return target;
	}

### Delete key from Tree
	public T delete(T key) throws EmptyTreeException {
		if(isEmpty())
			throw new EmptyTreeException("Empty Tree");
		
		if(search(key) == null)
		{
			//Not in tree
			return null;
		}
		root = delete_helper(root,key);

		return key;
		
		
	}
	
	public Node delete_helper(Node n, T key) {
		
		Node curr = null;
		
		if(n == null)
			return null;
		
		if(key.compareTo(n.value) < 0) {
			n.lnode = delete_helper(n.lnode, key);
		}
		else if(key.compareTo(n.value) > 0) {
			n.rnode = delete_helper(n.rnode, key);
		}
		else {
			
			//Delete this node
			if((n.lnode == null) && (n.rnode == null))
			{
				//Both children are null so just delete this
				curr = n;
				n = null;
				
			}
			else if((n.lnode != null) && (n.rnode == null)) {
				//Only left child
				n = n.lnode;
				
			}
			else if((n.lnode == null) && (n.rnode != null))
			{
				//Only right child
				n = n.rnode;
				
			}
			else {
				//In order successor
				curr = n.rnode;
				
				while (curr.lnode != null)
			        curr = curr.lnode;
				
				n.value = curr.value;
				n.height = curr.height;
				
				n.rnode = delete_helper(n.rnode, curr.value);
								
			}
		}

		if(n == null)
			return n;

		n.height = Math.max(heightHelper(n.lnode),heightHelper(n.rnode));
		
		int bal = getBalance(n);
		
		
	// Right Rotation
		if(n.lnode != null)
			if (bal > maxImbalance && (getBalance(n.lnode) >= 0)) {
					return rotateRight(n);
				}
		
		// Left Rotation
		if(n.rnode != null)
			if (bal < -1*maxImbalance && (getBalance(n.rnode) <= 0)) {
					return rotateLeft(n);
				}
		
		// Left Right Rotation
		if(n.lnode != null)
			if (bal > maxImbalance && (getBalance(n.lnode) < 0)) {
					n.lnode = rotateLeft(n.lnode);
					return rotateRight(n);
				}
		

		// Right Left Rotation
		if(n.rnode != null)
			if (bal < -1*maxImbalance && (getBalance(n.rnode) > 0)) {
					n.rnode = rotateRight(n.rnode);
					return rotateLeft(n);
				}
		

		return n;

	}

### Search for Key
	public T search(T key) throws EmptyTreeException {
		if (isEmpty())
			throw new EmptyTreeException("Empty Tree");

		boolean isThere = searchHelper(root, key);

		if (isThere)
			return key;
		else
			return null;
	}

	public boolean searchHelper(Node currNode, T key) {
		if (currNode == null)
			return false;
		else {
			if (currNode.value == key)
				return true;
			else {
				boolean a = false;
				boolean b = false;
				if(currNode.lnode != null) {
					a = searchHelper(currNode.lnode, key);
				}
				
				if(currNode.rnode != null)
					b = searchHelper(currNode.rnode, key);

				if (a || b)
					return true;
				else
					return false;
			}
		}
	}
  
### Gets Maximum Imbalance Factor
	public int getMaxImbalance() {
		return maxImbalance;
	}

### Gets Height of Tree, where root is a height of 0
	public int getHeight() {
		// Node is root which is empty
		if (root == null)
			return -1;
		else {
			return root.height;
		}
	}

	public int heightHelper(Node currNode) {
		if (currNode == null)
			return 0;
		else {
			// Current node is not null
			int lheight = 0;
			int rheight = 0;
			if (currNode.lnode != null) {
				lheight = heightHelper(currNode.lnode);
			}

			if (currNode.rnode != null) {
				rheight = heightHelper(currNode.rnode);
			}

			if (lheight > rheight) {
				return 1 + lheight;
			} else {
				return 1 + rheight;
			}

		}
	}


### Checks if tree is empty
	public boolean isEmpty() {
		return root == null;
	}

### Returns the root node
	public T getRoot() throws EmptyTreeException {
		if(isEmpty())
			throw new EmptyTreeException("Empty Tree");
		return root.value;

	}



### Check if the Tree satisfies the balance requirements
	public boolean isAVLGBalanced() {
		return ((Math.abs(getBalance(root)) <= maxImbalance));
	}

	public int getBalance(Node n) {
		if(n == null)
			return 0;
		
		int l = heightHelper(n.lnode);
		int r = heightHelper(n.rnode);
		return (l - r);

	}

### Empties the AVL Tree of all of its elements
	public void clear() {
		if(root == null)
			return;
		root.lnode = null;
		root.rnode = null;
		root.value = null;
		root.height = 0;
		root = null;
	}


### Returns number of elements in the tree
	public int getCount() {
		if(root == null)
			return 0;
		else
			return 1 + countHelper(root);
	}

	public int countHelper(Node currNode) {
		if (currNode == null)
			return 0;
		else {
			int t = 0;
			if (currNode.lnode != null) {
				t += 1 + countHelper(currNode.lnode);
			}
			if (currNode.rnode != null) {
				t += 1 + countHelper(currNode.rnode);
			}

			return t;
		}
	}
  }



