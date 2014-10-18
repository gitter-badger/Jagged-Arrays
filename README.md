/*
Jagged-Arrays
=============
Objective:
I have implemented a custom data structure called as JaggedArrays in C++. A JaggedArray is a 1D array of n bins, and each bin stores a collection of elements of templated type T. This data structure is helpful to ensure high performance for applications needing frequent read and write access to an organizational information. The JaggedArray class that is implemented has two fundamental representation modes: unpacked and packed. In the unpacked representation, the JaggedArray stores an array of integers containing the counts of how many elements are in each bin and an array of pointers to arrays that hold these elements. In the packed representation, the JaggedArray stores an array of offsets for the start of each bin. These offsets refer to a single large array that stores all of the elements grouped by bin but all packed into the same large array.  We can convert a JaggedArray from unpacked to packed mode by calling the pack member function and similarly convert from packed to unpacked by calling unpack. 

The member functions used for the implementation include: numElements, numBins, isPacked, addElement, removeElement, pack, unpack and print. */

#ifndef jagged_array_h_
#define jagged_array_h_
#include<iostream>
#include<fstream>
#include<cstdlib>
using namespace std;

template <class T> class JaggedArray {
public:
    //TYPEDEFS
    typedef unsigned int size_type;

    //CONSTRUCTORS, ASSIGNMENT OPERATORS AND DESTRUCTORS
    JaggedArray(const int& bins) { this->create(bins); }
    JaggedArray() { this->create(); }
    JaggedArray(const JaggedArray& v) { this->copy(v); }
    JaggedArray<T>& operator=(const JaggedArray<T>& v);
    ~JaggedArray();//DESTRUCTOR

    //ACCESSORS
    size_type numElements() const { return num_elements; }
    size_type numBins() const {return num_bins; }
    int numElementsInBin(int bin_no) const;
    T& getElement(int bin_no, int position_in_bin) const;
    bool isPacked() const;
    void print();
    void pack();
    void unpack();

    //MODIFIERS
    void addElement(int position, T element);
    void removeElement(int bin_pos, int pos_in_bin);
    void clear();

private:

    // PRIVATE MEMBER FUNCTIONS
    void create();
    void create(const int& bins);
    void copy(const JaggedArray<T>& v);

    // REPRESENTATION
    size_type num_elements;
    size_type num_bins;
    size_type *counts;
    T** unpacked_values;
    T* packed_values;
    size_type *offsets;

};

template <class T>  void JaggedArray<T>::create() {
//initialization of member variables
  counts = NULL;
  num_elements = num_bins = 0;
  unpacked_values=NULL;
  packed_values=NULL;
  offsets=NULL;
}

template <class T> void JaggedArray<T>::create(const int& bins) {
//constructor with argument as number of bins
    offsets=NULL;
    packed_values=NULL;
    num_elements=0;
    num_bins=bins;
    //creating arrays -counts, unpacked_values
    counts = new size_type [num_bins];
    for(int i=0; i<num_bins;i++)
    {
      counts[i]=0;
    }
    unpacked_values=new T*[bins];

    for(size_type i=0; i<num_bins;i++)
    {
        unpacked_values[i]=new T[0];
    }
}

template <class T> int JaggedArray<T>::numElementsInBin(int bin_no) const{

 if(bin_no<0 || bin_no>=num_bins)
    {
        cerr<<"cannot find this bin";
        exit(1);
    }
if(isPacked())
    {
        if(bin_no==num_bins-1)
            return num_elements-offsets[bin_no];
        else
            return offsets[bin_no+1]-offsets[bin_no];
    }
else
    return counts[bin_no];
}



template <class T> bool JaggedArray<T>::isPacked() const{



//checking if packed or unpacked
if(counts==NULL && unpacked_values==NULL)
    return true;
else
    return false;
}

template <class T> void JaggedArray<T>::addElement(int position, T element){

    if(!isPacked())
        {
            if(position>num_bins || position<0)
            {
            cerr<<"no such bin exists:";
            exit(1);
            }
            else
            {
            for(int i=0; i<num_bins; i++)
                {
                if(i==position)
                    {
                        T *new_arr= new T[numElementsInBin(i)+1]; 	//creating a new array with extra 1 space
                        for(int j=0;j<numElementsInBin(i);j++)
                        {
                            new_arr[j]=unpacked_values[i][j]; 		//transferring values to a temporary array
                        }
                if(unpacked_values!=NULL)
                    {
                    delete[] unpacked_values[i]; //deleting the old array
                    unpacked_values[i]=NULL;
                    }
                unpacked_values[i]=new_arr; 	//pointing the temp array to unpacked_values again which is now updated
                unpacked_values[i][numElementsInBin(i)]=element;
                counts[i]+=1; 			//increasing counters
                num_elements+=1;
                    }
                }
            }
        }

    else{
        cerr<<"Cannot add elements in packed mode :";
        exit(1);
    }
}




template <class T> T& JaggedArray<T>:: getElement(int bin_no, int position_in_bin)const{

    if(isPacked())
    {
        if(bin_no<0 || bin_no>(numBins()-1))        //Error checking
        {
        cerr<<"Incorrect inputs"<<endl;
        exit(1);
        }
        else if(position_in_bin>(numElementsInBin(bin_no)-1) || position_in_bin<0)
        {
        cerr<<"Incorrect inputs"<<endl;
        exit(1);
        }
        else{
        int offset=offsets[bin_no];
        return packed_values[offset+position_in_bin];   //calculating unpacked_values from packed values using offset
        }
    }

    else{
        if(bin_no<0 || bin_no>(numBins()-1))
        {
        cerr<<"Incorrect inputs"<<endl;
        exit(1);
        }

        else if(position_in_bin>(numElementsInBin(bin_no)-1) || position_in_bin<0)
        {
        cerr<<"Incorrect inputs"<<endl;
        exit(1);
        }
        else
        return unpacked_values[bin_no][position_in_bin];
    }
}




template <class T> void JaggedArray<T>::unpack(){

    if(isPacked())
    {
        unpacked_values=new T*[num_bins];
        counts = new size_type [num_bins];
        for(int i=0; i<num_bins;i++)
        {
            counts[i]=0;
        }
        for(int i=0;i<num_bins;i++)
        {
            if(i==num_bins-1)
                counts[i]=num_elements-offsets[i]; //calculating the elements in counts array using offsets and num_elements
            else
                counts[i]=offsets[i+1]-offsets[i];
        }
        for(int i=0;i<num_bins;i++)
            {
            unpacked_values[i]= new T[numElementsInBin(i)];
                for(int j=0; j<counts[i];j++)
                {
                        int offset=offsets[i];
                         unpacked_values[i][j]=packed_values[offset+j];
                }

            }
        if(offsets!=NULL)
        {
        delete [] offsets;
        offsets=NULL;
        }
        if(packed_values!=NULL)
        {
        delete [] packed_values;
        packed_values=NULL;
        }
    }

   else{
        cerr<<"Jagged Array already in unpacked mode"<<endl;
        exit(1);
   }

}



template <class T> void JaggedArray<T>::pack(){

    if(!isPacked())
    {
        int temp=0;
        int counter=1;
        if(packed_values==NULL)
        packed_values=new T[num_elements];
        if(offsets==NULL)
        offsets=new size_type [num_bins];
        for(int i=1; i<num_bins;i++)
        {
        offsets[0]=0;
        counter=i;
        while(counter>0)        //calculating the elements to be stored in offsets using counts
        {
            temp+=counts[counter-1];
            counter--;
        }
            offsets[i]=temp;
            temp=0;
            counter=1;
    }
    int counter1=0;
    for(unsigned int i=0; i<num_bins; i++)
        {
        for(unsigned int j=0; j<numElementsInBin(i);j++)
            {
                packed_values[counter1]=unpacked_values[i][j];  //copying the elements in unpacked to packed array
                counter1++;
            }
        }
    if(counts!=NULL)
     {
        delete [] counts;
        counts=NULL;
     }
   if(unpacked_values!=NULL)
     {
        for(int i=0; i<num_bins; i++)
          {
            if(unpacked_values[i]!=NULL)
                delete [] unpacked_values[i];
          }
        delete [] unpacked_values;
        unpacked_values=NULL;
     }
}
else{
    cerr<<"Jagged array is already in packed mode";
    exit(1);
    }

}
template <class T> JaggedArray<T>& JaggedArray<T>::operator=(const JaggedArray<T>& v){

    if (this != &v)
     {
        num_elements=v.numElements();
        num_bins=v.numBins();
        if(v.isPacked()==true)
        {
            unpacked_values=NULL;
            counts=NULL;
            delete [] offsets;
            delete [] packed_values;
            offsets=new size_type[num_bins];
            packed_values =new T[num_elements];
            for(int i=0; i<num_bins; i++)
                {
                    offsets[i]=v.offsets[i];
                }
            for(int j=0; j<num_elements; j++)
                {
                    packed_values[j]=v.packed_values[j];
                }
        }
    else
       {
        offsets=NULL;
        packed_values=NULL;
        delete[] counts;
        counts=NULL;
        for(int i=0; i<num_bins; i++)
            {
                delete [] unpacked_values[i];
                unpacked_values[i]=NULL;
            }
            delete [] unpacked_values;
            unpacked_values=NULL;
            counts=new size_type [num_bins];
            unpacked_values=new T*[num_bins];
            for(int i=0; i<num_bins; i++)
                {
                    counts[i]=v.numElementsInBin(i);
                }

            for(int i=0; i<num_bins; i++)
                {
                    unpacked_values[i]=new T[v.numElementsInBin(i)];
                    for(int j=0; j<v.numElementsInBin(i); j++)
                    unpacked_values[i][j]=v.getElement(i,j);
                }
        }

    }
  return *this;
}



template <class T> void JaggedArray<T>::copy(const JaggedArray<T>& v) {

        num_elements=v.numElements();
        num_bins=v.numBins();
        //copying counts and unpacked values for unpacked array
        if(v.isPacked()==false)
        {
        offsets=NULL;
        packed_values=NULL;
        counts = new size_type [num_bins];
        for(int i=0; i<this->num_bins;i++)
            {
                this->counts[i]=v.counts[i];
            }
        unpacked_values=new T*[num_bins];
        for(size_type i=0; i<this->num_bins;i++)
            {
                this->unpacked_values[i]= new T[numElementsInBin(i)];
                for(unsigned int j=0; j<this->numElementsInBin(i);j++)
                    {
                        this->unpacked_values[i][j]=v.getElement(i,j);
                    }
            }
        }
    else{
            //copying offsets and packed_values for packed array
            unpacked_values=NULL;
            counts=NULL;
            offsets = new size_type[num_bins];
            packed_values=new T [num_elements];
            for(int i=0; i<this->num_bins;i++)
            {
                this->offsets[i]=v.offsets[i];
            }

            for(int i=0; i<this->num_elements;i++)
            {
                this->packed_values[i]=v.packed_values[i];
            }

    }
}

template <class T>  void JaggedArray<T>::print() {
if(!isPacked())
{
cout<<"num_bins:  "<<numBins()<<std::endl;
cout<<"num_elements:  "<<numElements()<<std::endl;
cout<<"counts:  ";
for(int i=0;i<num_bins;i++)
    {
        std::cout<<numElementsInBin(i)<<' ';
    }
std::cout<<std::endl;
cout<<"values:  ";
for(unsigned int i=0; i<num_bins; i++)
    {
        for(unsigned int j=0; j<numElementsInBin(i);j++)
            {
            cout<<unpacked_values[i][j]<<' ';
            }
    }
    cout<<endl<<endl;
}

else{
    cout<<"num_bins:  "<<numBins()<<std::endl;
    cout<<"num_elements:  "<<numElements()<<std::endl;
    cout<<"offsets: ";
    for(int i=0;i<num_bins;i++)
        {
        std::cout<<offsets[i]<<' ';
        }
    cout<<endl;
    cout<<"values: ";

    for(int i=0;i<num_elements;i++)
        {
        std::cout<<packed_values[i]<<' ';
        }
cout<<endl<<endl;
    }
}


template <class T> void JaggedArray<T>:: removeElement(int bin_pos, int pos_in_bin)
{
    if(!isPacked())
    {
    if((bin_pos<0 && bin_pos>num_bins) || (pos_in_bin > numElementsInBin(bin_pos) && pos_in_bin<0))
        {
        cerr<<"Incorrect inputs"<<endl;
        }
    else
        {
        for(unsigned int j=pos_in_bin+1; j<numElementsInBin(bin_pos);j++)
            {
                unpacked_values[bin_pos][j-1]=unpacked_values[bin_pos][j];
            }
    num_elements--;
    counts[bin_pos]--;
        }
    }
    else{
        cerr<<"Cannot remove from a packed jagged array"<<endl;
        exit(1);
    }
}



template <class T>  void JaggedArray<T>::clear(){
if(!isPacked())
{
    num_elements=0;
    for(int i=0;i<numBins();i++)
    {
        counts[i]=0;//putting zeroes in counts for unpacked
        if(unpacked_values[i]!=NULL)
        {
            delete [] unpacked_values[i];
            unpacked_values[i]=NULL;
        }
    }
}

else{
    cerr<<"cannot clear a packed array"<<endl;
    exit(1);
}

}




template <class T> JaggedArray<T>::~JaggedArray()
{
    delete [] counts;
    delete [] offsets;
    delete [] packed_values;
    if(unpacked_values!=NULL)
        {
        for(int i=0; i<num_bins; i++)
            {
            delete [] unpacked_values[i];
            }
        delete [] unpacked_values;
        }
   counts=NULL;
   offsets=NULL;
   packed_values=NULL;
   unpacked_values=NULL;

}

#endif
